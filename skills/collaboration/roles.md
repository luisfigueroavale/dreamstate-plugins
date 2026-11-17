# Workspace roles

`workspace.membership_get` with `{}` returns the caller's `role` and `publishing_role`. `workspace.members_list` returns `current_user_role` and a page of members (`limit` 1 to 100). Page until the frontier is null before you claim someone is absent.

Workspace role on `workspace.update_member_role` is `admin`, `member`, `client`, or `viewer`. The call cannot make an owner. Ownership moves only with `workspace.transfer_ownership`, which is a separate explicit decision. Read the member first. The description says the call refuses an invalid target. Relay that code. Do not retry.

| Role | What the catalog states |
| --- | --- |
| owner | Not settable through `workspace.update_member_role` |
| admin | Can change workspace settings, invite, cancel invitations, and read billing |
| member | Can run workspace work |
| client | Can read, comment, and decide assigned approvals |
| viewer | Can see everything and change nothing |

Publishing role is separate: `manager`, `publisher`, `editor`, or `viewer`, changed with `workspace.member_publishing_role_update`. Content approvers need `publisher` or `manager`. Pass `expected_publishing_role` from the last read, or null when the stored value is null. A mismatch refuses the write. Do not substitute `manager` for a null stored role.

`workspace.invitation_create` takes `email` and `role` (`admin`, `member`, `client`, or `viewer`, default `member`). It sends an invitation email. It refuses the inviter's own address (`self_invitation`) and an existing member (`already_a_member`). Pending invitations are not members yet. Read `workspace.invitations_list` and do not report the role as held until `workspace.invitation_respond` accepts it.

You cannot satisfy a second-party approval by deciding your own request when decide returns 403. Invite the person who should decide, or leave the rule disabled until they are a publisher or manager. Do not archive a rule to bypass a review the user asked for.

API keys and connected apps act with their person's role. A refusal that names insufficient permission is final for this caller.
