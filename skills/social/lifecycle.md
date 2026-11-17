# Social lifecycle examples

The queue is the default. `social.posts.create` is only a LinkedIn text publish with no media and no schedule. Everything else is one artifact, then schedule, approve, and publish.

## Multi-channel draft

One request for LinkedIn and X is one `social.content.artifact_create`. LinkedIn reads `content`. X reads `platform_payloads.x.thread_data` and ignores `content`, so the first tweet does not have to equal the LinkedIn body.

```json
{
  "content": "The opener named the missed reply.\n\nThat change booked 4 meetings between 12 and 22 March.\n\nWhat would you change?",
  "channels": ["linkedin", "x"],
  "status": "draft",
  "platform_payloads": {
    "x": {
      "thread_data": [
        "Named the missed reply instead of the product.",
        "That opener booked 4 meetings from 12 to 22 March."
      ]
    }
  }
}
```

Read `social.content.artifact_get` with the returned `artifact_id`. `channels` has both ids, `platform_payloads.x.thread_data` still has two posts in that order, `status` is `draft`, and `deliveries` is empty. A 404 means the id is not in this workspace: list with `social.content.artifact_list` and do not create a second artifact.

## X poll on that same artifact shape

A poll is not a second draft. No media.

```json
{
  "content": "Which opener should we keep?",
  "channels": ["x"],
  "status": "draft",
  "platform_payloads": {
    "x": {
      "twitter_poll": {
        "options": ["Missed reply", "Product name"],
        "duration_minutes": 1440
      }
    }
  }
}
```

`social.content.artifact_get` must show both options and `duration_minutes` 1440. If create returns "Provider polls and quote posts cannot be combined with media.", drop the media and save once. Do not turn the poll into a quote to get past the refusal.

## Schedule refusal

`social.content.schedule` with `{artifact_id, expected_revision, scheduled_at}` writes deliveries and does not call a provider.

- 409 `account_selection_required`: the body names the accounts. `social.content.artifact_update` binds one `account_id` on that same artifact, then schedule again. Do not create another artifact.
- 400 `social_account_binding_required`: there is no channel, or no account and nothing to choose. Set the channel. `allow_unbound_draft` is only for a draft you are not scheduling.
- `provider_contract_not_found`: a LinkedIn schedule writes no delivery. Fix the account, then schedule the same artifact.
- `channel_media_required` or `channel_media_kind_unsupported`: attach or replace media, then schedule again.

`social.content.approve` then stamps `approved_version`. It publishes nothing. It blocks an empty body, LinkedIn text above 3,000 characters, a stored unresolved media reference, and every unresolved mention.

## Publish now

`social.content.delivery_publish` dry run returns `provider_payload` and `approved_publish_hash` and does not call the provider. Live publish sends that exact hash. The provider object is public when it commits. A row that is already published returns `replayed` true. `reconciliation_required` true means the outcome is unknown: read `social.content.artifact_get` and do not call `social.content.post_retry` until the delivery is `failed`.

`social.posts.create` previews a LinkedIn text post and publishes only when `mode` is `live` and `approved` is true. It cannot carry media or a time. A png, a thread, a poll, or a second channel stays on the queue above.

## Owned-post reply

`social.comments.reply` freezes one LinkedIn or X reply and returns `action_id` and `content_hash`. It does not post. `social.comments.approve` posts that preview when `approved` is true and the hash matches. An edit needs a new `social.comments.reply`. `effect_unknown` means read `social.comments.get` and the receipt. Do not approve or reply again for the same text.

`social.posts.comment_publish` is a different call: a new public comment on a live post, including Reddit, with no unpublish. It spends credits and follows the paid-run consent rule. Use it only when the comment is not already an owned-queue reply.
