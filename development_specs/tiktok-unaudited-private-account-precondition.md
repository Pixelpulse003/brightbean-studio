# TikTok unaudited private-account precondition

## Root cause

TikTok's unaudited_client_can_only_post_to_private_accounts error is not a
post-privacy mismatch. For an unaudited Content Posting API client, TikTok
requires both:

1. the target creator account itself to be private at posting time; and
2. the post viewership to be SELF_ONLY.

The composer persisted SELF_ONLY in PlatformPost.platform_extra, the
publisher copied it into PublishContent.extra, and the TikTok provider sent
it as post_info.privacy_level. TikTok still rejected the request because the
target creator account was public. BrightBean's previous error guidance
incorrectly told the operator to select “Only you” again, which made the
failure look like a persistence bug.

## Decision

Keep the existing privacy data flow unchanged and split the guidance for two
different failure classes:

- a creator-info option mismatch continues to instruct the operator to select
  SELF_ONLY;
- unaudited_client_can_only_post_to_private_accounts now explains that the
  target TikTok account visibility must also be private.

Regression coverage verifies the composer persistence layer, publisher
pass-through, provider payload, and the corrected permanent-error message.

## Operational validation

Before retrying an unaudited Direct Post:

1. verify the target TikTok account is private;
2. verify BrightBean persists privacy_level: SELF_ONLY;
3. obtain explicit operator confirmation immediately before publishing;
4. confirm the TikTok init payload is accepted before treating video.upload
   or video.publish as demonstrated.
