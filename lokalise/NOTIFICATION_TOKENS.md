# Notification tokens

Reference for the `[%x]` placeholders in `lokalise/notification-*.json`. Every string in these files is rendered server-side; the tokens below are the only dynamic values the copy can carry.

`notification-en.json` is the source of truth. All 54 locale files carry the same key tree and the same token set per key.

## The tokens

| Token | Means | Strings | Supplied by |
|---|---|---|---|
| `[%c]` | Creator / user display name | 53 | account |
| `[%p]` | Plan / package name | 22 | subscription |
| `[%s]` | Campaign name | 9 | campaign |
| `[%g]` | Date — grace period, expiry, or grant end | 7 | subscription / grant |
| `[%n]` | Number — supporter or new-supporter count | 4 | campaign stats |
| `[%r]` | Reason for an education grant revocation | 1 | moderation |

## Rules

- **Verbatim.** Case-sensitive, brackets included. `[%c]`, never `[%C]`, `{%c}`, or `[% c]`.
- **Never translate** the token, and never translate a word *inside* it — there is no word inside it.
- **Never pluralize** around a token by duplicating it. `[%n]` is a bare number; write copy that reads acceptably for 1 and for 200, or use an ICU plural in the main `<locale>.json` family instead.
- **Order is free.** Reposition tokens to fit the target grammar. Only the spelling is fixed.
- **Never drop a token `en` has.** If `en` carries two tokens for a key, every locale carries both. Dropping one silently loses the value from the rendered notification.
- **Extra tokens are allowed only if the server supplies that value for that notification.** A locale may name something `en` leaves implicit — see the divergence noted below — but a token the server does not pass for that notification renders empty.
- **Do not introduce `{0}`.** See below.

### Order is free, count is not

`[%c]` and `[%s]` may appear in whichever order the target language needs:

```
en     [%c] request approval to add a post on your campaign [%s]
ja     キャンペーン[%s]に投稿を追加するには、[%c]承認をリクエストしてください。
ko     [%s]캠페인에 게시물을 추가하기 위해 승인을 요청합니다 [%c]
```

`ja` and `ko` put the campaign first because their grammar wants it there. This is correct and expected. What is *not* allowed is dropping a token the source has.

### Known divergence

`id` names the plan in three `supporter.successful_payment` bodies where `en` leaves it implicit:

```
en   Enjoy your ad-free experience and get Twibbons without watermarks!
id   Dengan [%p], kamu bisa akses Twibbonize tanpa iklan ataupun watermark!
```

This is deliberate and renders correctly — these are billing notifications, so the server passes `[%p]`. The parity check below reports it as `EXTRA`, not as an error. Affected keys:

- `push.billing.supporter.successful_payment.body`
- `push.billing.international.supporter.successful_payment.body`
- `panel.billing.supporter.successful_payment.body`

## Why not `{0}`

These files previously used positional `{0}`. Seven strings carried **two different** `{0}` values — the creator name and the campaign name — in a single string:

```
{0} posted a Twibbon in your campaign {0}'s gallery!
```

Which `{0}` was which depended entirely on argument order, so a translator reordering the sentence for their language would silently swap a username and a campaign name. Named tokens make each slot self-describing and order-independent.

`{0}` no longer appears anywhere in `notification-*`. Do not reintroduce it.

## Other placeholder systems

Three systems coexist in this repo. Never convert one into another — match whatever the file you are editing already uses.

| System | Files | Examples |
|---|---|---|
| `[%x]` | `notification-*` | this document |
| `{name}` (ICU) | `<locale>.json` | `{plan}`, `{count}`, `{date}`, `{price}`, `{username}`, `{terms}` |
| `%s` / `%d` (printf) | `time-*` | `"%s ago"`, `"%d minutes"` |

In the main `<locale>.json` family, `{br}`, `{link}` and `{icon}` are markup slots — never wrap them, never duplicate them. That family also uses ICU plurals (`{count, plural, one{supporter} other{supporters}}`).

## Verifying

```sh
# no positional placeholders left
grep -l '{0}' lokalise/notification-*.json

# key parity and token parity against en
# DROPPED is always a bug. EXTRA is a copy choice - valid if the server
# passes that value for the notification (see "Known divergence" above).
python3 - <<'PY'
import json, glob, re, collections
def flat(o, p=''):
    if isinstance(o, dict):
        d = {}
        for k, v in o.items(): d.update(flat(v, p + '.' + k))
        return d
    return {p: o}
en = flat(json.load(open('lokalise/notification-en.json')))
def toks(s): return collections.Counter(re.findall(r'\[%[a-z]\]', s))
for f in sorted(glob.glob('lokalise/notification-*.json')):
    cur = flat(json.load(open(f)))
    for k, v in en.items():
        if k not in cur:
            print('MISSING KEY', f, k); continue
        dropped = toks(v) - toks(cur[k])
        extra = toks(cur[k]) - toks(v)
        if dropped: print('DROPPED', f, k, dict(dropped))
        if extra: print('EXTRA  ', f, k, dict(extra))
PY
```

## Full inventory

Generated from `notification-en.json`. A string appears under every token it contains.

#### `[%c]` — 53 strings

| Key | String |
|---|---|
| `push.campaign_milestone.header` | 🏆 [%c], new milestone reached! |
| `push.new_post.body` | [%c] posted a Twibbon in your campaign [%s]'s gallery! |
| `push.new_comment.body` | [%c] commented on your post in the campaign [%s]! |
| `push.account_recap.header` | 🍺 [%c], Here’s your monthly recap! |
| `push.billing.creator.plus_5.header` | Hi [%c], renew your subscription now |
| `push.billing.international.renewed_plan.header` | Hello [%c], your renewal payment has been confirmed |
| `push.billing.international.creator.plus_2.header` | Hi [%c], renew your subscription now |
| `push.billing.international.supporter.plus_2.header` | Hi [%c], renew your subscription now |
| `push.billing.supporter.plus_5.header` | Hi [%c], renew your subscription now |
| `push.new_reply.body` | [%c] replied on your comment in [%s] campaign! |
| `panel.campaign_milestone.header` | 🏆 [%c], new milestone reached! |
| `panel.account_recap.header` | 🍺 [%c], Here’s your monthly recap! |
| `panel.billing.order_received.header` | Hello [%c], please complete your payment |
| `panel.billing.order_canceled.header` | Sorry [%c], your [%p] order has been canceled |
| `panel.billing.creator.successful_payment.header` | Hello [%c], your payment has been received |
| `panel.billing.creator.min_3.header` | Hi [%c], your [%p] will expire in 3 days |
| `panel.billing.creator.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.creator.plus_5.header` | Hi [%c], your subscription will be ended in 48 hours |
| `panel.billing.creator.plus_7.header` | Hi [%c], your account is downgraded to Twibbonize Free |
| `panel.billing.international.creator.canceled_plan.header` | Hi [%c], your [%p] subscription was successfully canceled |
| `panel.billing.international.creator.renewed_plan.header` | Hi [%c], your [%p] subscription was successfully renewed |
| `panel.billing.international.creator.canceled.exp_day.header` | Hi [%c], your account has downgraded to Twibbonize Free |
| `panel.billing.international.creator.cancel_downgrade.header` | Hello [%c], your request to cancel the downgrade has been received |
| `panel.billing.international.creator.convert.header` | Hello [%c], Your account has been upgraded to Premium Creator |
| `panel.billing.international.creator.successful_payment.header` | Hello [%c], your payment has been received |
| `panel.billing.international.creator.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.international.creator.plus_2.header` | Hi [%c], your subscription will be ended in 48 hours |
| `panel.billing.international.creator.plus_5.header` | Hi [%c], your account has downgraded to Twibbonize Free |
| `panel.billing.international.creator.convert_addon.header` | Hello [%c], Your account has been upgraded to Premium Creator |
| `panel.billing.international.auth.exp_day.header` | Hi [%c], your payment method needs authentication |
| `panel.billing.international.renewed_plan.header` | Hello [%c], your renewal payment has been confirmed |
| `panel.billing.international.downgrade_received.header` | Hello [%c], your downgrade request has been received |
| `panel.billing.international.supporter.convert.header` | Hello [%c], Your account has been upgraded to Premium Supporter |
| `panel.billing.international.supporter.successful_payment.header` | Hi [%c], we have successfully received your payment |
| `panel.billing.international.supporter.exp_day.header` | Hello [%c], your payment didn't go through |
| `panel.billing.international.supporter.plus_2.header` | Hi [%c], your subscription will be ended in 48 hours |
| `panel.billing.international.supporter.plus_7.header` | Hi [%c], your account has downgraded to Twibbonize Free |
| `panel.billing.international.supporter.canceled.exp_day.header` | Hi [%c], your account has downgraded to Twibbonize Free |
| `panel.billing.international.supporter.plus_5.header` | Hi [%c], your account has downgraded to Twibbonize Free |
| `panel.billing.international.period_changed.header` | Hello [%c], we have received your request to change your plan |
| `panel.billing.international.cancel_change_period.header` | Hello [%c], we have received your request to cancel the plan change |
| `panel.billing.india.auth.exp_day.header` | Hi [%c], please confirm your payment for subscription now |
| `panel.billing.supporter.successful_payment.header` | Hello [%c], your payment has been received |
| `panel.billing.supporter.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.supporter.plus_5.header` | Hi [%c], your subscription will be ended in 48 hours |
| `panel.billing.supporter.min_3.header` | Hi [%c], your [%p] will expire in 3 days |
| `panel.billing.supporter.plus_7.header` | Hi [%c], your account is downgraded to Twibbonize Free |
| `panel.campaign_addon.adfree_quota_remaining.body` | You have Ad-Free quota remaining for 50 Supporters on [%c]. Enjoy an Ad-Free experience while it lasts! |
| `panel.campaign_addon.adfree_quota_runout.body` | Your [%c] supporters will start seeing ads while browsing on our platform. |
| `panel.new_post.body` | [%c] posted a Twibbon in your campaign [%s]'s gallery! |
| `panel.new_comment.body` | [%c] commented on your post in the campaign [%s]! |
| `panel.new_reply.body` | [%c] commented on your comment in the campaign [%s]! |
| `panel.approval.body` | [%c] request approval to add a post on your campaign [%s] |

#### `[%p]` — 22 strings

| Key | String |
|---|---|
| `panel.billing.order_received.body` | Thank you for ordering the [%p]. Please proceed to payment to complete your transaction. |
| `panel.billing.order_canceled.header` | Sorry [%c], your [%p] order has been canceled |
| `panel.billing.creator.successful_payment.body` | Thank you for subscribing to the [%p]. Create your campaign now and use its special features! |
| `panel.billing.creator.min_3.header` | Hi [%c], your [%p] will expire in 3 days |
| `panel.billing.creator.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.creator.plus_7.body` | Please renew your subscription to continue using [%p] features. |
| `panel.billing.international.creator.canceled_plan.header` | Hi [%c], your [%p] subscription was successfully canceled |
| `panel.billing.international.creator.canceled_plan.body` | You'll lose access to all [%p] features at the end of your billing cycle. |
| `panel.billing.international.creator.renewed_plan.header` | Hi [%c], your [%p] subscription was successfully renewed |
| `panel.billing.international.creator.renewed_plan.body` | Your [%p] subscription will be charged at end of your billing cycle. |
| `panel.billing.international.creator.canceled.exp_day.body` | Please renew your subscription to continue using [%p] features. |
| `panel.billing.international.creator.successful_payment.body` | Thank you for subscribing to the [%p]. Create your campaign now and use its special features! |
| `panel.billing.international.creator.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.international.creator.plus_5.body` | Please renew your subscription to continue using [%p] features. |
| `panel.billing.international.downgrade_received.body` | Your account will be downgraded to [%p] at the end of your billing cycle. |
| `panel.billing.international.supporter.successful.body` | Thank you for subscribing to the [%p]. Now, you can get Twibbon without watermarks and an ad-free experience! |
| `panel.billing.international.supporter.canceled.exp_day.body` | Please renew your subscription to continue using [%p] features. |
| `panel.billing.international.supporter.plus_5.body` | Please renew your subscription to continue using [%p] features. |
| `panel.billing.international.period_changed.body` | Your account will be changed to [%p] at the end of your billing cycle. |
| `panel.billing.supporter.exp_day.header` | Hello [%c], your [%p] subscription has expired |
| `panel.billing.supporter.min_3.header` | Hi [%c], your [%p] will expire in 3 days |
| `panel.billing.supporter.plus_7.body` | Please renew your subscription to continue using [%p] features. |

#### `[%s]` — 9 strings

| Key | String |
|---|---|
| `push.campaign_milestone.body` | Congrats! Your campaign [%s] now has [%n] supporters! |
| `push.new_post.body` | [%c] posted a Twibbon in your campaign [%s]'s gallery! |
| `push.new_comment.body` | [%c] commented on your post in the campaign [%s]! |
| `push.new_reply.body` | [%c] replied on your comment in [%s] campaign! |
| `panel.campaign_milestone.body` | Congratulations! The campaign [%s] now has [%n] supporters! |
| `panel.new_post.body` | [%c] posted a Twibbon in your campaign [%s]'s gallery! |
| `panel.new_comment.body` | [%c] commented on your post in the campaign [%s]! |
| `panel.new_reply.body` | [%c] commented on your comment in the campaign [%s]! |
| `panel.approval.body` | [%c] request approval to add a post on your campaign [%s] |

#### `[%g]` — 7 strings

| Key | String |
|---|---|
| `panel.billing.creator.exp_day.body` | Renew or upgrade your subscription before [%g] to maintain access to campaign analytics and all exclusive features. |
| `panel.billing.international.creator.exp_day.body` | Please update your billing information before [%g] to continue accessing your campaign analytics. |
| `panel.billing.international.supporter.exp_day.body` | Please update your billing information before [%g] to enjoy exclusive features, ad-free browsing and watermark-free Twibbon. |
| `panel.billing.supporter.exp_day.body` | Renew or upgrade your subscription before [%g] to continue enjoying exclusive features, ad-free browsing and watermark-free Twibbon. |
| `panel.education.d14_reminder.header` | Your Premium Creator for Education ends [%g] |
| `panel.education.grant_approved.body` | Active until [%g]. Start a campaign and try out your new features. |
| `panel.education.grant_extended.body` | Premium Creator is active until [%g]. |

#### `[%n]` — 4 strings

| Key | String |
|---|---|
| `push.campaign_milestone.body` | Congrats! Your campaign [%s] now has [%n] supporters! |
| `push.account_recap.body` | You gained [%n] new supporters this month. Keep it up! |
| `panel.campaign_milestone.body` | Congratulations! The campaign [%s] now has [%n] supporters! |
| `panel.account_recap.body` | You’ve gained [%n] new supporters in the past month across all your campaigns. Keep up the great work! |

#### `[%r]` — 1 strings

| Key | String |
|---|---|
| `panel.education.grant_revoked.body` | Reason: [%r]. Your account is back on Free. Tap to contact us. |

