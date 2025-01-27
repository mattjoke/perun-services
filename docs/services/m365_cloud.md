# m365_cloud

### [GEN](../concepts/gen.md)

The data is generated by the [generateMemberUsersDataInJson](../modules/PerunDataGenerator.md#generatememberusersdatainjson)
method, not valid VO members are filtered out. The data is additionally filtered, so that each user is assigned only to ONE license
group - the one with the highest priority. Prioritization of licences is expressed in the _m365AllowedLicensesPriorities_
facility attribute.

### [SEND](../concepts/send.md)
The data is sent to the Microsoft Graph API with obtained access token. Objects created by Perun
get PerunManaged extension attribute in M365 set to True. Users and groups not marked by this M365 attribute should stay intact.
Removed objects are stored in M365 for ~30 days and can be restored (NEED to be restored in case of renewed internal users as
they hold unique immutable ID, which would block creating the same user again).

#### Users Management
Each Perun user can be propagated as multiple accounts to M365. One account can be internal (fully manageable
in the target tenant) - the UPN is in format _login@scope_; and multiple accounts can be external (already existing
in other Microsoft tenants) - UPN is in format _formattedEmail#EXT#@scope.onmicrosoft.com_. External accounts are created
from existing user's Microsoft mails (_mails-namespace:microsoft_), internal accounts are only created if the user
is assigned to any license group. Both types of accounts are removed from m365 in case they are no longer part of
any Perun managed group. If user has no existing Microsoft mail and no internal identity can be added to group,
the user is skipped and returned in the propagation overview.

#### Groups Management
Perun resource can represent Team, Security group or Licensed group. Teams and Security groups are filled with all user's
external accounts and existing internal account (does not create new internal account), unless specified otherwise
by the _m365InternalAccountsOnly_ attribute (then only existing internal accounts are added to the Team/Group).
The Licensed groups are filled with internal accounts, and if it does not yet exist for the user, it is created first.
There are typically limited licenses acquired for each license type in the tenant, however, if this number is exceeded
due to Perun propagation, we don't get informed in the response.