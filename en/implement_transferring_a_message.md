# Implement Transferring a Message

Open **OpenChatChattingViewController.m** in Xcode.

![Open OpenChatChattingViewController.m](img/005_OpenChatChattingViewController_m.png)

The UI for chat is consist of ```UITableView``` for messages, ```UIButton``` for sending an image, ```UITextField``` for entering a message and ```UIButton``` for sending a message. 

Insert following code to log in and query the channel list into the bottom of ```viewDidLoad``` method.

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    
    …

    [Jiver loginWithUserId:[Jiver deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    [Jiver joinChannel:[channel url]];
    [Jiver setEventHandlerConnectBlock:^(JiverChannel *channel) {
        
    } errorBlock:^(NSInteger code) {
        
    } channelLeftBlock:^(JiverChannel *channel) {
        
    } messageReceivedBlock:^(JiverMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } systemMessageReceivedBlock:^(JiverSystemMessage *message) {
        
    } broadcastMessageReceivedBlock:^(JiverBroadcastMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } fileReceivedBlock:^(JiverFileLink *fileLink) {
        if (lastMessageTimestamp < [fileLink getMessageTimestamp]) {
            lastMessageTimestamp = [fileLink getMessageTimestamp];
        }
        
        if ([fileLink isPast]) {
            [messages insertObject:fileLink atIndex:0];
        }
        else {
            [messages addObject:fileLink];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } messagingStartedBlock:^(JiverMessagingChannel *channel) {
        
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
    
    [[Jiver queryMessageListInChannel:[channel url]] prevWithMessageTs:LLONG_MAX andLimit:50 resultBlock:^(NSMutableArray *queryResult) {
        for (JiverMessage *message in queryResult) {
            if ([message isPast]) {
                [messages insertObject:message atIndex:0];
            }
            else {
                [messages addObject:message];
            }
            
            if (lastMessageTimestamp < [message getMessageTimestamp]) {
                lastMessageTimestamp = [message getMessageTimestamp];
            }
        }
        [self scrollToBottomWithReloading:YES animated:NO];
        
        [Jiver connectWithMessageTs:LLONG_MAX];
    } endBlock:^(NSError *error) {
        
    }];
}
```