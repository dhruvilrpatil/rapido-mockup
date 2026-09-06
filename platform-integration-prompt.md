# Platform Integration Prompt

You are a senior Supabase and frontend engineer building a separate platform integration for `[PLATFORM_NAME]`.

Use the existing GigFolio architecture and Supabase project without breaking existing functionality. Do not delete, rename, or change existing tables, columns, RPC functions, RLS policies, data, UI layouts, or frontend contracts unless absolutely required.

## Supabase Connection

Use `@supabase/supabase-js`.

Use these environment variables:

```text
VITE_SUPABASE_URL=https://kblhngnyyaxphzecftet.supabase.co
VITE_SUPABASE_ANON_KEY=sb_publishable_SZK2GACanKrb4iI9j0KfVQ_Bq51EW8p
```

Never use or expose a Supabase service-role key in frontend code.

## Login

Support login using either a Gig ID or worker UUID.

- For Gig ID login, find the worker using `users.gig_id`.
- For UUID login, find the worker using `users.user_id`.

After login, load only the required public profile fields:

- `user_id`
- `gig_id`
- `legal_name`, if permitted
- `gig_score`
- `tier`
- `total_reviews`

## Separate Platform and GigFolio Ratings

The new platform must have its own independent platform rating. Never use `users.gig_score` as the platform rating.

Use this RPC to load the platform-specific rating:

```text
get_worker_platform_rating(p_user_id, p_platform_name)
```

Call it with:

```text
p_platform_name = '[PLATFORM_NAME]'
```

The RPC should return:

- `user_id`
- `platform_name`
- `platform_score`
- `tier`
- `total_reviews`

The platform score must be calculated only from reviews belonging to the same worker and the same platform.

The UI must display these values separately:

- `[PLATFORM_NAME] Rating`
- `[PLATFORM_NAME] Tier`
- `[PLATFORM_NAME] Review Count`
- `GigFolio Score`
- `GigFolio Tier`
- Overall GigFolio review count

The GigFolio score must come from `users.gig_score` or the existing GigFolio RPC.

The `[PLATFORM_NAME]` score must come from `get_worker_platform_rating`.

Never use the GigFolio score as the platform score.

## Review Submission

Preserve the existing 1–5 star review form.

When a review is submitted, call the existing `add_user_rating` RPC:

```js
const { data, error } = await supabase.rpc('add_user_rating', {
  p_user_id: workerUserId,
  p_rating: Number(selectedRating),
  p_review_text: reviewText || null,
  p_platform_name: '[PLATFORM_NAME]',
  p_reviewer_name: reviewerName || 'Anonymous'
});
```

The exact platform name must be stored with the review. For example, if the platform is Rapido, `platform_name` must be stored as `Rapido`.

A review from `[PLATFORM_NAME]` must update:

- The combined GigFolio score
- The combined GigFolio tier
- The combined GigFolio review count
- The `[PLATFORM_NAME]` platform score
- The `[PLATFORM_NAME]` platform tier
- The `[PLATFORM_NAME]` platform review count

A review from another platform must:

- Update the combined GigFolio score
- Update the combined GigFolio tier
- Update the combined GigFolio review count
- Update that other platform's score and count
- Not update the `[PLATFORM_NAME]` score or count

## Post-Submission Updates

The existing `add_user_rating` RPC should return:

```json
{
  "success": true,
  "user_id": "uuid",
  "new_gig_score": 4.85,
  "tier": "Diamond Top Performer",
  "total_reviews": 4
}
```

After successful submission:

1. Update the GigFolio score with `new_gig_score`.
2. Update the GigFolio tier with `tier`.
3. Update the overall review count with `total_reviews`.
4. Call `get_worker_platform_rating` again.
5. Update the platform score with `platform_score`.
6. Update the platform tier with the returned `tier`.
7. Update the platform review count with the returned `total_reviews`.
8. Refresh the platform-specific recent reviews.
9. Never use `new_gig_score` as the platform score.

## Recent Reviews and Platform Breakdown

If available, call:

```js
const { data: recentReviews } = await supabase.rpc(
  'get_platform_recent_reviews',
  {
    p_user_id: workerUserId,
    p_platform_name: '[PLATFORM_NAME]',
    p_limit: 5
  }
);
```

Also call:

```js
const { data: platforms } = await supabase.rpc(
  'get_worker_platform_breakdown',
  {
    p_user_id: workerUserId
  }
);
```

Only show reviews belonging to the selected platform in the platform-specific review section.

Each platform must be displayed independently, including `[PLATFORM_NAME]`.

## UX Requirements

Preserve all existing:

- Loading states
- Error states
- Toast notifications
- Form inputs
- Success messages
- Responsive behavior
- Layout
- Styling
- Colors
- Design elements

Show loading feedback while logging in, loading profile data, and submitting a review.

Disable the submit button while the review is being saved.

Never show a success message when a Supabase RPC returns an error or `success` is false.

Show clear errors for:

- Invalid Gig IDs
- Invalid UUIDs
- Failed profile loading
- Failed platform-rating loading
- Failed review submission
- Missing platform data

Show an empty state when the platform has no reviews.

Do not show private fields such as email, phone number, authentication tokens, or service-role credentials.

## Database Requirements

First inspect the existing schema and definitions for:

- `users`
- The existing review table
- `add_user_rating`
- `get_worker_platform_rating`
- `get_platform_recent_reviews`
- `get_recent_reviews`
- `get_worker_platform_breakdown`

Preserve the existing database architecture.

Every review must store:

- `user_id`
- `rating`
- `platform_name`
- `review_text`
- `reviewer_name`
- `created_at`

If `platform_name` does not exist in the current review table, add it using a non-destructive migration while preserving all existing rows.

Use case-insensitive platform matching:

```sql
lower(platform_name) = lower(p_platform_name)
```

Validate that ratings are between `1.00` and `5.00`.

Reject blank platform names.

Use transaction-safe database logic.

Preserve existing RLS policies and allow the frontend to execute only the required safe RPCs.

Do not expose private fields such as email, phone number, authentication tokens, or service-role keys.

## Platform Tier Rules

- Score >= 4.80: Diamond Top Performer
- Score >= 4.50: Gold Verified
- Score >= 4.00: Silver Active
- Score < 4.00: Bronze Starter

## Testing Requirements

Test login with:

- A valid Gig ID
- A valid worker UUID
- An invalid Gig ID
- An invalid UUID

Test review submission with:

- A 1-star review
- A 3-star review
- A 5-star review
- A review from `[PLATFORM_NAME]`
- A review from another platform

Confirm that:

- The review stores the exact platform name.
- The platform rating updates separately.
- The GigFolio rating updates separately.
- A review from another platform does not change `[PLATFORM_NAME]` rating.
- Platform review counts are separate from overall review counts.
- The platform breakdown lists `[PLATFORM_NAME]` independently.
- The UI never displays the GigFolio score as the platform score.
- Existing platforms and existing functionality remain unchanged.

## Deliverables

Return:

1. The changed frontend files.
2. The Supabase RPCs used.
3. Any database migration required.
4. Confirmation that Gig ID and UUID login both work.
5. Confirmation that platform and GigFolio ratings are stored and displayed separately.
6. Build, lint, and test results.
7. Confirmation that the existing architecture and unrelated platform behavior were preserved.
