# Implement Group Messaging

Group messaging is the extened concept of the 1-on-1 messaging. It has the feature for inviting users, typing indicator and unread message count.


Three or more users is required in order to start group messaging. The sample project provides the feature for it in Messaging tab.

You can select users who you want to invite in Messaging **Tab > Invite > Select Channel > Select User**. 

![Messaging Tab](img/010_Screenshot.png) ![Select Channel](img/011_Screenshot.png) ![Select User](img/012_Screenshot.png)


## Invite Users

The callback block is required to get the messaging channel after inviting users.

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    isLoadingUser = NO;
    
    [self.messagingInviteSelectUserTableView setContentInset:UIEdgeInsetsMake(64, 0, 0, 0)];
    [self.messagingInviteSelectUserTableView setDelegate:self];
    [self.messagingInviteSelectUserTableView setDataSource:self];
    
    userArray = [[NSMutableArray alloc] init];
    
    [Jiver loginWithUserId:[Jiver deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    memberListQuery = [Jiver queryMemberListInChannel:[selectedChannel url]];
    [memberListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
        for (JiverMember *user in queryResult) {
            [userArray addObject:user];
        }
        [self.messagingInviteSelectUserTableView reloadData];
    } endBlock:^(NSError *error) {

    }];
    [Jiver setEventHandlerConnectBlock:^(JiverChannel *channel) {
        
    } errorBlock:^(NSInteger code) {
        
    } channelLeftBlock:^(JiverChannel *channel) {
        
    } messageReceivedBlock:^(JiverMessage *message) {
        
    } systemMessageReceivedBlock:^(JiverSystemMessage *message) {
        
    } broadcastMessageReceivedBlock:^(JiverBroadcastMessage *message) {
        
    } fileReceivedBlock:^(JiverFileLink *fileLink) {
        
    } messagingStartedBlock:^(JiverMessagingChannel *channel) {
        UIStoryboard *storyboard = [self storyboard];
        MessagingViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@"MessagingViewController"];
        [vc setMessagingChannel:channel];
        [vc setDelegate:self];
        [self presentViewController:vc animated:YES completion:nil];
    } messagingUpdatedBlock:^(JiverMessagingChannel *channel) {
        
    } messagingEndedBlock:^(JiverMessagingChannel *channel) {
        
    } allMessagingEndedBlock:^{
        
    } messagingHiddenBlock:^(JiverMessagingChannel *channel) {
        
    } allMessagingHiddenBlock:^{
        
    } readReceivedBlock:^(JiverReadStatus *status) {
        
    } typeStartReceivedBlock:^(JiverTypeStatus *status) {
        
    } typeEndReceivedBlock:^(JiverTypeStatus *status) {
        
    } allDataReceivedBlock:^(NSUInteger jiverDataType, int count) {
        
    } messageDeliveryBlock:^(BOOL send, NSString *message, NSString *data, NSString *messageId) {
        
    }];
}
```

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

