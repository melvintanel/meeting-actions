# Meeting actions

A small tracker for actions agreed in meetings. Add an action with an owner and due date, group by owner, tick off what's done, and see open, done and overdue counts. Team members sign in, and data is stored in Supabase so everyone sees the same list.

## Files

- `index.html`: the whole app (HTML, CSS and JavaScript in one file)
- `vercel.json`: static hosting settings and basic security headers

## Sign-in and users

People sign in with a username and password. Passwords are stored by Supabase Auth in `auth.users`. The app maps each username to a hidden email address (`username@meetingactions.example.com`), because Supabase Auth signs in with an email.

Starting users (initial password `password`, change it after first sign-in):

| Username | Role |
| --- | --- |
| melvin | Admin (can add users) |
| serene | Member |
| daniel | Member |

Admins see an Add user button, where they set the username and initial password. Everyone can change their own password.

## Supabase tables

- `auth.users`: built-in Supabase Auth table, holds the accounts and hashed passwords
- `public.profiles`: links each auth user to a username, display name and admin flag
- `public.meeting_actions`: the actions, with `created_by` pointing at the auth user who added them
- `public.reminders`: each person's own reminders (message, time, optional linked action, sent time)
- `public.user_phone_alerts`: each person's private ntfy topic, never readable by other users

Row level security means only signed-in team members (a user with a `profiles` row) can read or change actions. The functions `admin_create_user`, `is_member` and `is_admin` do the user management and permission checks.

The Supabase URL and publishable key are set at the top of the script in `index.html`.

## Reminders and phone alerts

Anyone signed in can add reminders (the Reminders button, or the bell on any action). Alerts reach phones through [ntfy](https://ntfy.sh), a free push notification service:

1. Each person installs the ntfy app (iPhone or Android) and subscribes to their private topic, shown in the Reminders panel.
2. A Supabase scheduled job (`pg_cron`, every minute) finds reminders that are due and sends them to that person's topic using `pg_net`. They arrive at urgent priority, even with the phone locked.
3. The Send test alert button checks the setup.

Notes:
- Reminders fire within about a minute of the chosen time.
- The topic name works like a password. Anyone who knows it can read that person's alerts, and the reminder text passes through ntfy.sh. Keep sensitive details out of reminders, or use Get a new topic if one leaks.
- To change how alerts are sent (for example SMS), edit the `send_due_reminders` function in Supabase.

## Recommended Supabase dashboard settings

- Authentication, Sign In / Providers, Email: turn off Allow new users to sign up. The app already ignores accounts without a profile, but this stops strangers creating accounts at all.
- Authentication, Policies: leave the minimum password length at 6 or higher.

## Deploy on Vercel

1. In Vercel, choose Add New, then Project, and import this repo.
2. Set Framework Preset to Other. Leave Build Command and Output Directory empty.
3. Click Deploy.

Every push to the main branch redeploys automatically.
