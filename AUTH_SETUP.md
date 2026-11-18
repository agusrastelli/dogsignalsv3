# Email authentication — staged implementation

This branch replaces simulated email accounts with Supabase magic-link authentication. It is intentionally staged until outbound email is configured and a real end-to-end email test passes.

## Configuration

- Supabase project: `gnyeqxytxhdksrdyulpp`.
- Site URL was changed to `https://dogsignalsv3.netlify.app/` on November 18, 2026.
- The client requests this exact URL as its callback.
- Keep email confirmation enabled. Anonymous sessions remain available for the chat's guest mode.
- Configure a custom SMTP provider in Authentication > Emails > SMTP Settings. The default Supabase sender is limited to project-team addresses, so it cannot serve general participants.
- Enter SMTP credentials only in Supabase, never in this repository or the frontend.

## Scope and storage

- Successful email submission shows a waiting screen, never a signed-in state.
- Identity is established by Supabase's email confirmation session. Initial sessions are checked with `getUser`.
- Sign-in does not create new users; registration can create them.
- Guest data and verified user data use separate local storage namespaces. Existing prototype data migrates into the guest namespace only. Old prototype emails are never trusted as real identities.
- After explicit registration and verification of the matching email within one hour, guest profile/chat/progress may be carried into that user's local namespace. Existing local user data is not overwritten.
- Profile, chat and progress are still local to the browser. This branch does NOT implement cross-device cloud sync or a multiple-conversation archive. UI copy states this limitation.
- Local storage is not encrypted and is not a security boundary against other people or scripts with access to the same browser profile. Private cloud storage will require owner-bound database policies.
- New chat still clears the current conversation. Switching identity invalidates pending chat responses.
- Authentication callback fragments are excluded from research entry logs.

## Validation

Run `node tests/auth.test.cjs`.

Covers JavaScript syntax, anonymous/unverified users, account isolation, guest preservation, email-link waiting state, SMTP errors, registration carryover and sign-out failures/successes with mocked Supabase responses.

Before merging, test on the configured callback domain:
1. Register a permitted test email and confirm there is no signed-in state before opening the link.
2. Open the email, confirm the correct identity and expected profile.
3. Reload, sign out, sign back in, then test a second verified user. No chats should cross accounts.
4. Test expired links and unavailable email service.
5. Verify chat calls work for both guest and verified sessions.
6. Confirm local-only storage copy and New chat behavior are clear on desktop/mobile.

A real email has not been sent or verified during this implementation. Finish SMTP setup and coordinate a test email with the owner before merging.
