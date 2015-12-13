# Implement Group Messaging

Group messaging is the extened concept of the 1-on-1 messaging. It has the feature for inviting users, typing indicator and unread message count.


Three or more users is required in order to start group messaging. The sample project provides the feature for it in Messaging tab.

You can select users who you want to invite in Messaging **Tab > Invite > Select Channel > Select User**. 

![Messaging Tab](img/010_Screenshot.png) ![Select Channel](img/011_Screenshot.png) ![Select User](img/012_Screenshot.png)


## Invite Users



![MessagingInviteSelectUserViewController.m](img/013_MessagingInviteSelectUserViewController_m.png)


You have to modify ```inviteUsers:``` method in order to invite multiple users.

```objectivec
- (IBAction)inviteUsers:(id)sender {
    NSArray * indexPaths = [self.messagingInviteSelectUserTableView indexPathsForSelectedRows];
    NSMutableArray<NSString *> *userIds = [[NSMutableArray alloc] init];
    for (NSIndexPath *indexPath in indexPaths) {
        [userIds addObject:[[userArray objectAtIndex:indexPath.row] guestId]];
    }
    
    if ([userIds count] > 0) {
        [Jiver startMessagingWithUserIds:userIds];
    }
}
```