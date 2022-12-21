# openchat

Open-source chat app built on [PocketBase](https://pocketbase.io)

Required PocketBase schema can be found in `./pocketbase/pb_schema.json`

## Features

- Individual channels
  - Channel invites
  - Basic channel permissions (owner/member)
  - Message sending
    - Attachments supported
  - Manage invites
  - Manage messages (must be sender to delete own, only owner can delete any message)
- User profiles
  - Badges (can be assigned by admin only)
  - User avatar
  - User bio (soon)
