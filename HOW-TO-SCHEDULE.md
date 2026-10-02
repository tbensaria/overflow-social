# Daily Buffer top-up (Overflow Social Media Hell Month)

`posts.json` is the source of truth. Each day has captions per platform, a media slot and a
`scheduled` record. Buffer's free plan holds at most 10 scheduled posts, so only the next
3 days (9 posts) are ever queued.

For each day from today to today+2 (Europe/London):

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
