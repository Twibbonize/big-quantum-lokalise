# Notification catalogue

Every notification string in `lokalise/notification-*.json`, generated from `notification-en.json`.

For what the `[%x]` tokens mean and the rules around them, see [NOTIFICATION_TOKENS.md](NOTIFICATION_TOKENS.md).

## Structure

```
notification
├── push      delivered as a device push notification
└── panel     shown in the in-app notification panel
```

Each notification is an object with `header` and `body`. A handful carry only one of the two — that is intentional and matches what the server renders; do not add the missing half without a product decision.

`header` is the bold line. `body` is the supporting line beneath it.

Nesting under `billing` reflects who is billed and where:

| Path segment | Means |
|---|---|
| `creator` | Premium Creator subscriber |
| `supporter` | Premium Supporter subscriber |
| `international` | non-Indonesian payment flow (Stripe) |
| `india` | India-specific recurring-payment mandate flow |
| `auth` | payment needs cardholder authentication |
| `canceled` | subscription already cancelled, now lapsing |

Lifecycle suffixes:

| Suffix | Fires |
|---|---|
| `min_3`, `min_1` | 3 days / 1 day **before** the event |
| `exp_day` | on the expiry date |
| `plus_2`, `plus_5`, `plus_7` | 2 / 5 / 7 days **after** expiry |

## Education

`panel.education` covers the Premium Creator for Education grant lifecycle. It has no `push` counterpart — these are panel-only.

| Key | Fires when | Tap target |
|---|---|---|
| `d14_reminder` | 14 days before the grant expires | Settings → Billing |
| `grant_expired_extendable` | grant lapsed, extension still available | Settings → Billing |
| `grant_expired_not_extendable` | grant lapsed, four-year cap reached | Pricing page |
| `grant_approved` | grant approved | Start a Campaign modal |
| `grant_revoked` | grant revoked by moderation | twibbonize.com/contact-us |
| `grant_extended` | grant extended by 12 months | Settings → Billing |

Tap targets are routing metadata and are **not** stored in the locale files — they live in the client. Listed here so the copy can be read against its destination.

The two `grant_expired_*` headers are deliberately identical in every locale; they differ only in body copy and tap target.

`grant_revoked` uses `[%r]`, which no other notification uses. See the token doc for its backend status.

## Adding a notification

1. Add it to `notification-en.json` first, inside the section it belongs to.
2. Add the same key path to all 53 other `notification-*.json` files with a real translation.
3. Keep the token set identical to `en` for each key — same tokens, same count. Order may change to fit the grammar.
4. Regenerate the tables below.
5. Do not put a UI string here; those belong in `<locale>.json`.

## Regenerating this document

The tables below are generated. To refresh them after adding or editing a notification:

```sh
python3 - <<'PY'
import json, re, collections
d = json.load(open('lokalise/notification-en.json'),
              object_pairs_hook=collections.OrderedDict)['notification']

def walk(node, path=''):
    for k, v in node.items():
        if not isinstance(v, dict): continue
        leaf = ('header' in v or 'body' in v) and all(not isinstance(x, dict) for x in v.values())
        p = path + '.' + k if path else k
        if leaf: yield p, v
        else: yield from walk(v, p)

for fam in ('push', 'panel'):
    entries = list(walk(d[fam]))
    print('## `%s` — %d notifications\n' % (fam, len(entries)))
    print('| Key | Header | Body | Tokens |')
    print('|---|---|---|---|')
    for k, e in entries:
        h = e.get('header', '—').replace('|', '\\|')
        b = e.get('body', '—').replace('|', '\\|')
        t = sorted(set(re.findall(r'\[%[a-z]\]', e.get('header','') + ' ' + e.get('body',''))))
        print('| `%s` | %s | %s | %s |' % (k, h, b, ' '.join('`%s`' % x for x in t) or '—'))
    print()
PY
```

## `push` — 14 notifications

| Key | Header | Body | Tokens |
|---|---|---|---|
| `campaign_milestone` | 🏆 [%c], new milestone reached! | Congrats! Your campaign [%s] now has [%n] supporters! | `[%c]` `[%n]` `[%s]` |
| `new_post` | New Twibbon posted | [%c] posted a Twibbon in your campaign [%s]'s gallery! | `[%c]` `[%s]` |
| `new_comment` | New comment! | [%c] commented on your post in the campaign [%s]! | `[%c]` `[%s]` |
| `account_recap` | 🍺 [%c], Here’s your monthly recap! | You gained [%n] new supporters this month. Keep it up! | `[%c]` `[%n]` |
| `billing.creator.successful_payment` | Your payment was successful | Take advantage of its exclusive features! Start your campaign now. | — |
| `billing.creator.plus_5` | Hi [%c], renew your subscription now | Renew or upgrade your plan now to continue accessing your campaign analytics. | `[%c]` |
| `billing.international.renewed_plan` | Hello [%c], your renewal payment has been confirmed | Now you can continue use our exclusive features! | `[%c]` |
| `billing.international.creator.successful_payment` | Your payment was successful | Take advantage of its exclusive features! Start your campaign now. | — |
| `billing.international.creator.plus_2` | Hi [%c], renew your subscription now | Renew or upgrade your plan now to continue accessing your campaign analytics. | `[%c]` |
| `billing.international.supporter.successful_payment` | Your payment was successful | Enjoy your ad-free experience and get Twibbons without watermarks! | — |
| `billing.international.supporter.plus_2` | Hi [%c], renew your subscription now | Renew your plan now to enjoy watermark-free Twibbons and an ad-free experience. | `[%c]` |
| `billing.supporter.successful_payment` | Your payment was successful | Enjoy your ad-free experience and get Twibbons without watermarks! | — |
| `billing.supporter.plus_5` | Hi [%c], renew your subscription now | Renew your plan now to enjoy watermark-free Twibbons and an ad-free experience. | `[%c]` |
| `new_reply` | New reply! | [%c] replied on your comment in [%s] campaign! | `[%c]` `[%s]` |

## `panel` — 53 notifications

| Key | Header | Body | Tokens |
|---|---|---|---|
| `campaign_milestone` | 🏆 [%c], new milestone reached! | Congratulations! The campaign [%s] now has [%n] supporters! | `[%c]` `[%n]` `[%s]` |
| `account_recap` | 🍺 [%c], Here’s your monthly recap! | You’ve gained [%n] new supporters in the past month across all your campaigns. Keep up the great work! | `[%c]` `[%n]` |
| `billing.order_received` | Hello [%c], please complete your payment | Thank you for ordering the [%p]. Please proceed to payment to complete your transaction. | `[%c]` `[%p]` |
| `billing.order_canceled` | Sorry [%c], your [%p] order has been canceled | Please do not continue to payment. | `[%c]` `[%p]` |
| `billing.creator.successful_payment` | Hello [%c], your payment has been received | Thank you for subscribing to the [%p]. Create your campaign now and use its special features! | `[%c]` `[%p]` |
| `billing.creator.min_3` | Hi [%c], your [%p] will expire in 3 days | Renew or upgrade your subscription to continue enjoying Twibbonize's exclusive features. | `[%c]` `[%p]` |
| `billing.creator.exp_day` | Hello [%c], your [%p] subscription has expired | Renew or upgrade your subscription before [%g] to maintain access to campaign analytics and all exclusive features. | `[%c]` `[%g]` `[%p]` |
| `billing.creator.plus_5` | Hi [%c], your subscription will be ended in 48 hours | Renew or upgrade your plan within 48 hours to maintain access to your campaign analytics. | `[%c]` |
| `billing.creator.plus_7` | Hi [%c], your account is downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `billing.international.creator.canceled_plan` | Hi [%c], your [%p] subscription was successfully canceled | You'll lose access to all [%p] features at the end of your billing cycle. | `[%c]` `[%p]` |
| `billing.international.creator.renewed_plan` | Hi [%c], your [%p] subscription was successfully renewed | Your [%p] subscription will be charged at end of your billing cycle. | `[%c]` `[%p]` |
| `billing.international.creator.canceled.exp_day` | Hi [%c], your account has downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `billing.international.creator.cancel_downgrade` | Hello [%c], your request to cancel the downgrade has been received | Your subscription will continue as Premium Creator. | `[%c]` |
| `billing.international.creator.convert` | Hello [%c], Your account has been upgraded to Premium Creator | Congratulations! You are now a Premium Creator. Enjoy all of the features now! | `[%c]` |
| `billing.international.creator.successful_payment` | Hello [%c], your payment has been received | Thank you for subscribing to the [%p]. Create your campaign now and use its special features! | `[%c]` `[%p]` |
| `billing.international.creator.exp_day` | Hello [%c], your [%p] subscription has expired | Please update your billing information before [%g] to continue accessing your campaign analytics. | `[%c]` `[%g]` `[%p]` |
| `billing.international.creator.plus_2` | Hi [%c], your subscription will be ended in 48 hours | Please update your payment card to maintain access to your campaign analytics. | `[%c]` |
| `billing.international.creator.plus_5` | Hi [%c], your account has downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `billing.international.creator.convert_addon` | Hello [%c], Your account has been upgraded to Premium Creator | Congratulations! You are now a Premium Creator. Enjoy all of the features now! | `[%c]` |
| `billing.international.auth.exp_day` | Hi [%c], your payment method needs authentication | Please confirm your payment authentication to complete your subscription now. | `[%c]` |
| `billing.international.renewed_plan` | Hello [%c], your renewal payment has been confirmed | Now, you can continue enjoying exclusive features! | `[%c]` |
| `billing.international.downgrade_received` | Hello [%c], your downgrade request has been received | Your account will be downgraded to [%p] at the end of your billing cycle. | `[%c]` `[%p]` |
| `billing.international.supporter.convert` | Hello [%c], Your account has been upgraded to Premium Supporter | Congratulations! You are now a Premium Supporter. Enjoy all of the features now! | `[%c]` |
| `billing.international.supporter.successful_payment` | Hi [%c], we have successfully received your payment | Now, you can get Twibbon without watermarks and an ad-free experience! | `[%c]` |
| `billing.international.supporter.min_3` | — | Renew or upgrade your subscription now to continue enjoying ad-free browsing and watermark-free Twibbons. | — |
| `billing.international.supporter.successful` | — | Thank you for subscribing to the [%p]. Now, you can get Twibbon without watermarks and an ad-free experience! | `[%p]` |
| `billing.international.supporter.exp_day` | Hello [%c], your payment didn't go through | Please update your billing information before [%g] to enjoy exclusive features, ad-free browsing and watermark-free Twibbon. | `[%c]` `[%g]` |
| `billing.international.supporter.plus_2` | Hi [%c], your subscription will be ended in 48 hours | Renew your plan now to enjoy watermark-free Twibbons and an ad-free experience. | `[%c]` |
| `billing.international.supporter.plus_7` | Hi [%c], your account has downgraded to Twibbonize Free | — | `[%c]` |
| `billing.international.supporter.canceled.exp_day` | Hi [%c], your account has downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `billing.international.supporter.plus_5` | Hi [%c], your account has downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `billing.international.period_changed` | Hello [%c], we have received your request to change your plan | Your account will be changed to [%p] at the end of your billing cycle. | `[%c]` `[%p]` |
| `billing.international.cancel_change_period` | Hello [%c], we have received your request to cancel the plan change | Your subscription will continue as Premium Creator. | `[%c]` |
| `billing.india.auth.exp_day` | Hi [%c], please confirm your payment for subscription now | To ensure compliance with security regulations, we require your explicit confirmation for your recurring payment on your Twibbonize subscription. | `[%c]` |
| `billing.supporter.successful_payment` | Hello [%c], your payment has been received | Now, you can get Twibbon without watermarks and an ad-free experience! | `[%c]` |
| `billing.supporter.exp_day` | Hello [%c], your [%p] subscription has expired | Renew or upgrade your subscription before [%g] to continue enjoying exclusive features, ad-free browsing and watermark-free Twibbon. | `[%c]` `[%g]` `[%p]` |
| `billing.supporter.plus_5` | Hi [%c], your subscription will be ended in 48 hours | Renew your plan within 48 hours to keep enjoying watermark-free Twibbon and an ad-free experience. | `[%c]` |
| `billing.supporter.min_3` | Hi [%c], your [%p] will expire in 3 days | Renew or upgrade your subscription now to continue enjoying ad-free browsing and watermark-free Twibbons. | `[%c]` `[%p]` |
| `billing.supporter.plus_7` | Hi [%c], your account is downgraded to Twibbonize Free | Please renew your subscription to continue using [%p] features. | `[%c]` `[%p]` |
| `launch.prem_sub.min_1` | New Twibbonize Premium Subscription | Introducing Twibbonize's New Premium Subscriptions - Launching July 8! | — |
| `launch.short_link.min_1` | New Twibbonize Short Link | Introducing Twibbonize's New Short Link - Changing July 8! | — |
| `campaign_addon.adfree_quota_remaining` | Ad-Free quota remaining | You have Ad-Free quota remaining for 50 Supporters on [%c]. Enjoy an Ad-Free experience while it lasts! | `[%c]` |
| `campaign_addon.adfree_quota_runout` | Ad-Free quota has run out | Your [%c] supporters will start seeing ads while browsing on our platform. | `[%c]` |
| `new_post` | New Twibbon posted | [%c] posted a Twibbon in your campaign [%s]'s gallery! | `[%c]` `[%s]` |
| `new_comment` | New comment! | [%c] commented on your post in the campaign [%s]! | `[%c]` `[%s]` |
| `new_reply` | New reply! | [%c] commented on your comment in the campaign [%s]! | `[%c]` `[%s]` |
| `approval` | Post Needs Approval | [%c] request approval to add a post on your campaign [%s] | `[%c]` `[%s]` |
| `education.d14_reminder` | Your Premium Creator for Education ends [%g] | Extend before then to keep your Premium Creator features. | `[%g]` |
| `education.grant_expired_extendable` | Your education grant has ended | You're back on Free. Extend to get your Premium Creator features back. | — |
| `education.grant_expired_not_extendable` | Your education grant has ended | You're back on Free. Purchase Premium Creator to keep your Creator features. | — |
| `education.grant_approved` | You've got Premium Creator, free | Active until [%g]. Start a campaign and try out your new features. | `[%g]` |
| `education.grant_revoked` | Your education grant has been revoked | Reason: [%r]. Your account is back on Free. Tap to contact us. | `[%r]` |
| `education.grant_extended` | Your grant is extended 12 months | Premium Creator is active until [%g]. | `[%g]` |

