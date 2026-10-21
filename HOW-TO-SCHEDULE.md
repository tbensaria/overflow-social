# Daily Buffer top-up (Overflow Social Media Hell Month)

`posts.json` is the source of truth. Each day has captions per platform, a media slot and a
`scheduled` record. Buffer's free plan holds at most 10 scheduled posts, so the queue is filled
whole days at a time until the next day would push it past 10 (3 days, 9 posts, in practice).
The live count comes from Buffer (`list_posts`), not from `posts.json`.

For each day from today onwards (Europe/London), in date order:

1. Skip it if `hold` is set (it needs a human choice) or its media slot is empty. Report it.
2. For each platform whose `scheduled.<platform>` is null and whose Buffer channel exists:
   - X: `x.text`, asset `media.x` (video, GIF or image per `x.type`).
   - TikTok: `tiktok.text`, video `media.vertical`.
   - YouTube: `youtube.title` (metadata.youtube.title), `youtube.text`, categoryId "20"
     (Gaming), privacy public, video `media.vertical`.
   - `mode: customScheduled`, `dueAt` = `<date>T17:00:00+01:00` (BST until 25 Oct 2026,
     then `+00:00`), `schedulingType: automatic`.
3. Write the returned Buffer post id into `scheduled.<platform>`, commit and push.

Reddit is not part of the Hell Month (dropped 2026-10-02). YouTube posts are added automatically once the channel is connected to Buffer.

## Guard rails (added 2026-10-21)

- **One run at a time.** The task takes a lock file (`run.lock` beside its SKILL.md) and skips if a
  run younger than 3 hours holds it. Overlapping runs double-booked Days 18 and 19.
- **Check Buffer before every create.** A post is never created if Buffer already has one for that
  channel on that London date. Ids are committed day by day.
- **Missed days move to the end.** A day whose 17:00 slot passed without posting is never posted
  late; it gets `movedFrom` and the next free date after the last day. Days 16 and 17 moved to
  2 and 3 Nov this way.
