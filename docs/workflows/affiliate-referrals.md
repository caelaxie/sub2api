# Affiliate referrals

The affiliate workflow lets a user invite others, earn rebate quota, and
transfer available affiliate quota into normal balance.

```text
inviter shares link
  |
  +-- new user registers with aff code
        |
        +-- later eligible activity creates rebate
        |
        +-- inviter transfers available quota to balance
```

## User flow

1. The affiliate page loads the user's affiliate code, invite link, invitees,
   rebate statistics, and available/frozen quota.
2. The user copies the referral code or registration URL.
3. Registration and OAuth registration preserve affiliate codes.
4. Transfer moves available affiliate quota into user balance.
5. The frontend refreshes affiliate data and current user state after transfer.

## Admin flow

1. Admins can inspect invite records.
2. Admins can inspect rebate records.
3. Admins can inspect transfer records.
4. Admins can configure affiliate settings for specific users.

## Important rules

- Transfer is disabled when available quota is zero or negative.
- Backend errors are surfaced through the normal API error path.
- Affiliate codes are preserved across regular registration and OAuth signup
  helpers.

## Source map

- `backend/internal/server/routes/user.go`
- `backend/internal/server/routes/admin.go`
- `backend/internal/handler/user_handler.go`
- `backend/internal/handler/admin/affiliate_handler.go`
- `backend/internal/service/affiliate_service.go`
- `frontend/src/views/user/AffiliateView.vue`
- `frontend/src/views/admin/affiliates/AdminAffiliateInvitesView.vue`
- `frontend/src/views/admin/affiliates/AdminAffiliateRebatesView.vue`
- `frontend/src/views/admin/affiliates/AdminAffiliateTransfersView.vue`

