# Implement 1-on-1 Messaging

1-on-1 messaging requires two users. Inteage identifies each user by User ID. The sample project provides a feature for 1-on-1 messaging where a user can invite another user by obtaining ***User ID*** in the open chat channel.

## Start Messaging

You need target's ***User ID*** in order to start 1-on-1 messaging. Each message in an open chat contains sender's ***User ID***, which can be used to create 1-on-1 messaging.

Open **OpenChatChattingViewController.m** in Xcode.

![OpenChatChattingViewController.m](img/004_OpenChatChattingViewController_m.png)

Implement [tableView:didSelectRowAtIndexPath:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableViewDelegate_Protocol/#//apple_ref/occ/intfm/UITableViewDelegate/tableView:didSelectRowAtIndexPath:) to obtain the ***User ID*** of the message's sender.

```objectivec
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    UIAlertController *messageSubMenu;
    UIAlertAction *messageAction;
    UIAlertAction *messageCancelAction;

    if ([[messages objectAtIndex:indexPath.row] isKindOfClass:[InteageMessage class]]) {
        InteageMessage *message = (InteageMessage *)[messages objectAtIndex:indexPath.row];
        
        if ([[[message sender] guestId] isEqualToString:[Inteage getUserId]]) {
            return;
        }
        
        NSString *actionTitle = [NSString stringWithFormat:@"Start messaging with %@", [message getSenderName]];
        messageSubMenu = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];
        messageAction = [UIAlertAction actionWithTitle:actionTitle style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            [Inteage startMessagingWithUserId:[[message sender] guestId]];
        }];
        messageCancelAction = [UIAlertAction actionWithTitle:@"Cancel" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {}];
        [messageSubMenu addAction:messageAction];
        [messageSubMenu addAction:messageCancelAction];
        
        [self presentViewController:messageSubMenu animated:YES completion:nil];
    }
    else {
        return;
    }
}
```

You can see ```UIAlertController``` when you click other's messages in the open chat like this:

![UIAlertController for starting messaging](img/007_Screenshot.png)


If you click "Start messaging with OPPONENT_NAME", then [Inteage startMessagingWithUserId:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/startMessagingWithUserId:) is invoked. When it is invoked, the callback [messagingStartedBlock:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/setEventHandlerConnectBlock:errorBlock:channelLeftBlock:messageReceivedBlock:systemMessageReceivedBlock:broadcastMessageReceivedBlock:fileReceivedBlock:messagingStartedBlock:messagingUpdatedBlock:messagingEndedBlock:allMessagingEndedBlock:messagingHiddenBlock:allMessagingHiddenBlock:readReceivedBlock:typeStartReceivedBlock:typeEndReceivedBlock:allDataReceivedBlock:messageDeliveryBlock:) will be invoked as well. The callback returns [InteageMessagingChannel](http://docs.inteage.com/ref/ios/en/Classes/InteageMessagingChannel.html) object, which allows the user to join the 1-on-1 messaging channel.

You should insert codes for joining a channel in [messagingStartedBlock:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/setEventHandlerConnectBlock:errorBlock:channelLeftBlock:messageReceivedBlock:systemMessageReceivedBlock:broadcastMessageReceivedBlock:fileReceivedBlock:messagingStartedBlock:messagingUpdatedBlock:messagingEndedBlock:allMessagingEndedBlock:messagingHiddenBlock:allMessagingHiddenBlock:readReceivedBlock:typeStartReceivedBlock:typeEndReceivedBlock:allDataReceivedBlock:messageDeliveryBlock:). Refer to the codes in ```messagingStartedBlock:``` for details.
 
```objectivec
- (void)startChattingWithPreviousMessage:(BOOL)tf
{
    [Inteage loginWithUserId:[Inteage deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    [Inteage joinChannel:[currentChannel url]];
    [Inteage setEventHandlerConnectBlock:^(InteageChannel *channel) {
        
    } errorBlock:^(NSInteger code) {
        
    } channelLeftBlock:^(InteageChannel *channel) {
        
    } messageReceivedBlock:^(InteageMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [message getMessageTimestamp]) {
            firstMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } systemMessageReceivedBlock:^(InteageSystemMessage *message) {
        
    } broadcastMessageReceivedBlock:^(InteageBroadcastMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [message getMessageTimestamp]) {
            firstMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } fileReceivedBlock:^(InteageFileLink *fileLink) {
        if (lastMessageTimestamp < [fileLink getMessageTimestamp]) {
            lastMessageTimestamp = [fileLink getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [fileLink getMessageTimestamp]) {
            firstMessageTimestamp = [fileLink getMessageTimestamp];
        }
        
        if ([fileLink isPast]) {
            [messages insertObject:fileLink atIndex:0];
        }
        else {
            [messages addObject:fileLink];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
    } messagingStartedBlock:^(InteageMessagingChannel *channel) {
        UIStoryboard *storyboard = [self storyboard];
        MessagingViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@"MessagingViewController"];
        [vc setMessagingChannel:channel];
        [vc setDelegate:self];
        [self presentViewController:vc animated:YES completion:nil];
    } messagingUpdatedBlock:^(InteageMessagingChannel *channel) {
        
    } messagingEndedBlock:^(InteageMessagingChannel *channel) {
        
    } allMessagingEndedBlock:^{
        
    } messagingHiddenBlock:^(InteageMessagingChannel *channel) {
        
    } allMessagingHiddenBlock:^{
        
    } readReceivedBlock:^(InteageReadStatus *status) {
        
    } typeStartReceivedBlock:^(InteageTypeStatus *status) {
        
    } typeEndReceivedBlock:^(InteageTypeStatus *status) {
        
    } allDataReceivedBlock:^(NSUInteger inteageDataType, int count) {
        
    } messageDeliveryBlock:^(BOOL send, NSString *message, NSString *data, NSString *messageId) {
        
    }];
    
    if (tf) {
        [[Inteage queryMessageListInChannel:[currentChannel url]] prevWithMessageTs:LLONG_MAX andLimit:50 resultBlock:^(NSMutableArray *queryResult) {
            for (InteageMessage *message in queryResult) {
                if ([message isPast]) {
                    [messages insertObject:message atIndex:0];
                }
                else {
                    [messages addObject:message];
                }
                
                if (lastMessageTimestamp < [message getMessageTimestamp]) {
                    lastMessageTimestamp = [message getMessageTimestamp];
                }
                
                if (firstMessageTimestamp > [message getMessageTimestamp]) {
                    firstMessageTimestamp = [message getMessageTimestamp];
                }
                
            }
            [self scrollToBottomWithReloading:YES animated:NO];
            scrollLocked = NO;
            [Inteage connectWithMessageTs:LLONG_MAX];
        } endBlock:^(NSError *error) {
            
        }];
    }
    else {
        [Inteage connect];
    }
}
```

These lines of code get a view controller for messaging from the storyboard and the view controller is set with the channel returned from the block. As a result, the view controller is initialized.

## Implement Messaging

```MessagingViewController.m``` is invoked by ```messagingStartedBlock:``` in ```OpenChatChattingViewController.m```. Open ```MessagingViewController.m``` in Xcode to implement messaging features including a message transfer, a typing indicator and an unread message count. When the current channel is updated ```registerNotificationHandlerMessagingChannelUpdatedBlock:``` will be invoked, then you have to update the infomation and message's read status of the channel.

![MessagingViewController.m](img/008_MessagingViewController_m.png)

Modify ```viewDidLoad``` method to initialze the messaging system.

```objectivec
- (void)viewDidLoad {
    // ...
    
    [Inteage loginWithUserId:[Inteage deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    [Inteage registerNotificationHandlerMessagingChannelUpdatedBlock:^(JiverMessagingChannel *channel) {
        if ([Inteage getCurrentChannel] != nil && [[Inteage getCurrentChannel] channelId] == [channel getId]) {
            [self updateMessagingChannel:channel];
        }
    }
    mentionUpdatedBlock:^(InteageMention *mention) {
       
    }];
    [Inteage setEventHandlerConnectBlock:^(InteageChannel *channel) {
        [Inteage markAsRead];
    } errorBlock:^(NSInteger code) {
        
    } channelLeftBlock:^(InteageChannel *channel) {
        
    } messageReceivedBlock:^(InteageMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [message getMessageTimestamp]) {
            firstMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
        
        [Inteage markAsRead];
    } systemMessageReceivedBlock:^(InteageSystemMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [message getMessageTimestamp]) {
            firstMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
        
        [Inteage markAsRead];
    } broadcastMessageReceivedBlock:^(InteageBroadcastMessage *message) {
        if (lastMessageTimestamp < [message getMessageTimestamp]) {
            lastMessageTimestamp = [message getMessageTimestamp];
        }
        
        if (firstMessageTimestamp > [message getMessageTimestamp]) {
            firstMessageTimestamp = [message getMessageTimestamp];
        }
        
        if ([message isPast]) {
            [messages insertObject:message atIndex:0];
        }
        else {
            [messages addObject:message];
        }
        [self scrollToBottomWithReloading:YES animated:NO];
        
        [Inteage markAsRead];
    } fileReceivedBlock:^(InteageFileLink *fileLink) {
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
        
        [Inteage markAsRead];
    } messagingStartedBlock:^(InteageMessagingChannel *channel) {
        currentChannel = channel;
        [self updateMessagingChannel:channel];
        
        [[Inteage queryMessageListInChannel:[currentChannel getUrl]] prevWithMessageTs:LLONG_MAX andLimit:50 resultBlock:^(NSMutableArray *queryResult) {
            for (InteageMessage *message in queryResult) {
                if ([message isPast]) {
                    [messages insertObject:message atIndex:0];
                }
                else {
                    [messages addObject:message];
                }
                
                if (lastMessageTimestamp < [message getMessageTimestamp]) {
                    lastMessageTimestamp = [message getMessageTimestamp];
                }
                
                if (firstMessageTimestamp > [message getMessageTimestamp]) {
                    firstMessageTimestamp = [message getMessageTimestamp];
                }
            }
            [self scrollToBottomWithReloading:YES animated:NO];
            [Inteage joinChannel:[currentChannel getUrl]];
            scrollLocked = NO;
            [Inteage connectWithMessageTs:LLONG_MAX];
        } endBlock:^(NSError *error) {
            
        }];
    } messagingUpdatedBlock:^(InteageMessagingChannel *channel) {
        currentChannel = channel;
        [self updateMessagingChannel:channel];
    } messagingEndedBlock:^(InteageMessagingChannel *channel) {
        
    } allMessagingEndedBlock:^{
        
    } messagingHiddenBlock:^(InteageMessagingChannel *channel) {
        
    } allMessagingHiddenBlock:^{
        
    } readReceivedBlock:^(InteageReadStatus *status) {
        [self setReadStatus:[[status user] guestId] andTimestamp:[status timestamp]];
        [self.messagingTableView reloadData];
    } typeStartReceivedBlock:^(InteageTypeStatus *status) {
        [self setTypeStatus:[[status user] guestId] andTimestamp:[status timestamp]];
        [self showTyping];
    } typeEndReceivedBlock:^(InteageTypeStatus *status) {
        [self setTypeStatus:[[status user] guestId] andTimestamp:0];
        [self showTyping];
    } allDataReceivedBlock:^(NSUInteger inteageDataType, int count) {
        
    } messageDeliveryBlock:^(BOOL send, NSString *message, NSString *data, NSString *messageId) {
        
    }];
    [Inteage joinMessagingWithChannelUrl:[currentChannel getUrl]];
}
```

We must manage the timestamp of the last and the first message. The timestamp of the last message will be used for loading next messages and the timestamp of the first will be used for loading the previous messages.

In the above code we used LLONG_MAX value for [prevWithMessageTs:](http://docs.inteage.com/ref/ios/en/Classes/InteageMessageListQuery.html#//api/name/prevWithMessageTs:andLimit:resultBlock:endBlock:) which means that the latest messages can be fetched from Inteage server. The messaging channel, however, can receive new message while it is fetching other messages from the server. So we invoke [[Inteage connectWithMessageTs:LLONG_MAX]](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/connectWithMessageTs:) in ```resultBlock:```. Any new message will be passed to ```messageReceivedBlock:``` of ```[Inteage setEventHandlerConnectBlock:...]``` like as a real-time message.

## Load Previous Messages

Modify ```loadPreviosMessage``` method to fetch previous messages.

```objectivec
- (void) loadPreviousMessages {
    if (isLoadingMessage) {
        return;
    }
    isLoadingMessage = YES;
    
    [self.prevMessageLoadingIndicator setHidden:NO];
    [self.prevMessageLoadingIndicator startAnimating];
    [[Inteage queryMessageListInChannel:[currentChannel getUrl]] prevWithMessageTs:firstMessageTimestamp andLimit:50 resultBlock:^(NSMutableArray *queryResult) {
        NSMutableArray *newMessages = [[NSMutableArray alloc] init];
        for (InteageMessage *message in queryResult) {
            if ([message isPast]) {
                [newMessages insertObject:message atIndex:0];
            }
            else {
                [newMessages addObject:message];
            }
            
            if (lastMessageTimestamp < [message getMessageTimestamp]) {
                lastMessageTimestamp = [message getMessageTimestamp];
            }
            
            if (firstMessageTimestamp > [message getMessageTimestamp]) {
                firstMessageTimestamp = [message getMessageTimestamp];
            }
        }
        NSUInteger newMsgCount = [newMessages count];

        if (newMsgCount > 0) {
            [messages insertObjects:newMessages atIndexes:[NSIndexSet indexSetWithIndexesInRange:NSMakeRange(0, newMsgCount)]];
            
            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
                [self.messagingTableView reloadData];
                if ([newMessages count] > 0) {
                    [self.messagingTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:([newMessages count] - 1) inSection:0] atScrollPosition:UITableViewScrollPositionTop animated:NO];
                }
                isLoadingMessage = NO;
                [self.prevMessageLoadingIndicator setHidden:YES];
                [self.prevMessageLoadingIndicator stopAnimating];
            });
        }
        else {
            isLoadingMessage = NO;
            [self.prevMessageLoadingIndicator setHidden:YES];
            [self.prevMessageLoadingIndicator stopAnimating];
        }
    } endBlock:^(NSError *error) {
        isLoadingMessage = NO;
        [self.prevMessageLoadingIndicator setHidden:YES];
        [self.prevMessageLoadingIndicator stopAnimating];
    }];
}
```
## Send a Message to a Channel

To send a message, modify ```sendMessage:``` method. This method is invoked by clicking the ***Send*** button or pressing the return key. [Inteage sendMessage:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/sendMessage:) method sends message in real-time.

```objectivec
- (void) sendMessage
{
    NSString *message = [self.messageTextField text];
    if ([message length] > 0) {
        [self.messageTextField setText:@""];
        [Inteage sendMessage:message];
        [Inteage typeEnd];
    }
    scrollLocked = NO;
}
```

If you click the ***File*** button, ```clickSendFileButton:``` method will be invoked to open [UIImagePickerController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImagePickerController_Class/). Since [UIImagePickerController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImagePickerController_Class/) is used to select an image to send, modify the following method - [Inteage uploadFile:type:hasSizeOfFile:withCustomField:uploadBlock:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/uploadFile:type:hasSizeOfFile:withCustomField:uploadBlock:) which uploads ```imageFileData``` to Inteage server. This method returns [InteageFileInfo](http://docs.inteage.com/ref/ios/en/Classes/InteageFileInfo.html) object which can be sent through Inteage [Inteage sendFile:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/sendFile:).

```objectivec
- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
{
    __block NSString *mediaType = [info objectForKey: UIImagePickerControllerMediaType];
    __block UIImage *originalImage, *editedImage, *imageToUse;
    __block NSURL *imagePath;
    __block NSString *imageName;
    
    [picker dismissViewControllerAnimated:YES completion:^{
        if (CFStringCompare ((CFStringRef) mediaType, kUTTypeImage, 0) == kCFCompareEqualTo) {
            editedImage = (UIImage *) [info objectForKey:
                                       UIImagePickerControllerEditedImage];
            originalImage = (UIImage *) [info objectForKey:
                                         UIImagePickerControllerOriginalImage];
            
            if (originalImage) {
                imageToUse = originalImage;
            } else {
                imageToUse = editedImage;
            }
            
            NSData *imageFileData = UIImagePNGRepresentation(imageToUse);
            imagePath = [info objectForKey:@"UIImagePickerControllerReferenceURL"];
            imageName = [imagePath lastPathComponent];
            
            [Inteage uploadFile:imageFileData type:@"image/jpg" hasSizeOfFile:[imageFileData length] withCustomField:@"" uploadBlock:^(InteageFileInfo *fileInfo, NSError *error) {
                openImagePicker = NO;
                [Inteage sendFile:fileInfo];
            }];
        }
    }];
}
```

## Implement Typing Indicator

### Send Typing Status

The Messaging supports a typing indicator. The typing indicator shows who is typing and how many people are typing now.

When a message is being entered in ```UITextField```, send a command to notify typing status. When ```UITextField``` is empty, send a command to notify the end of typing.

```objectivec
- (void) textFieldDidChange:(UITextView *)textView
{
    if ([[textView text] length] > 0) {
        [Inteage typeStart];
    }
    else {
        [Inteage typeEnd];
    }
}
```

### Receive Typing Status

You can receive the typing status of other users in the same channel in callback blocks. ```[Inteage setEventHandlerConnectBlock:...``` includes these callbacks. 

```objectivec
    //...
    } typeStartReceivedBlock:^(InteageTypeStatus *status) {
        [self setTypeStatus:[[status user] guestId] andTimestamp:[status timestamp]];
        [self showTyping];
    } typeEndReceivedBlock:^(InteageTypeStatus *status) {
        [self setTypeStatus:[[status user] guestId] andTimestamp:0];
        [self showTyping];
    }
    //...
```

We always have to consider the network issue. The command that notifies the end of typing can get lost, so you need to implement a timer to handle this exception. The timer will remove the indicator after 10 seconds even if the command that notifies the end of typing is not received.

```objectivec
- (void)startTimer
{
    if (typingIndicatorTimer != nil) {
        [typingIndicatorTimer invalidate];
    }

    typingIndicatorTimer = [NSTimer scheduledTimerWithTimeInterval:10 target:self selector:@selector(clearTypingIndicator:) userInfo:nil repeats:NO];
}

- (void)clearTypingIndicator:(NSTimer *)timer
{
    [self hideTyping];
}
```

```objectivec
- (void) showTyping
{
    if ([typeStatus count] == 0) {
        [self hideTyping];
    }
    else {
        [self.typingIndicatorView setHidden:NO];
        [self.typeStatusLabel setHidden:NO];
        self.typingIndicatorHeight.constant = 48;
        [self.view updateConstraints];
        
        [self scrollToBottomWithReloading:NO animated:NO];
        
        [self.typeStatusLabel setText:[MyUtils generateTypingStatus:typeStatus]];
    }
    [self startTimer];
}
```

The typing indicator is at the bottom of the message table view.

![Typing Indicator](img/008_Screenshot.png)

## Manage Unread Count on Each Message

### Send Mark-as-Read

In order to display the number of users who did not read a certain message, you have to send the mark-as-read command whenever you receive message.

```objectivec
[Inteage setEventHandlerConnectBlock:^(InteageChannel *channel) {
        [Inteage markAsRead];
    } errorBlock:^(NSInteger code) {
        
    } channelLeftBlock:^(InteageChannel *channel) {
        
    } messageReceivedBlock:^(InteageMessage *message) {
        // ...
        [Inteage markAsRead];
    } systemMessageReceivedBlock:^(InteageSystemMessage *message) {
        // ...
        [Inteage markAsRead];
    } broadcastMessageReceivedBlock:^(InteageBroadcastMessage *message) {
        // ...
        [Inteage markAsRead];
    } fileReceivedBlock:^(InteageFileLink *fileLink) {
        // ...
        [Inteage markAsRead];
    } messagingStartedBlock:^(InteageMessagingChannel *channel) {
        // ...
    } messagingUpdatedBlock:^(InteageMessagingChannel *channel) {
        // ...
    } messagingEndedBlock:^(InteageMessagingChannel *channel) {
        // ...
    } allMessagingEndedBlock:^{
        // ...
    } messagingHiddenBlock:^(InteageMessagingChannel *channel) {
        // ...
    } allMessagingHiddenBlock:^{
        // ...
    } readReceivedBlock:^(InteageReadStatus *status) {
        // ...
    } typeStartReceivedBlock:^(InteageTypeStatus *status) {
        // ...
    } typeEndReceivedBlock:^(InteageTypeStatus *status) {
        // ...
    } allDataReceivedBlock:^(NSUInteger inteageDataType, int count) {
        // ...
    } messageDeliveryBlock:^(BOOL send, NSString *message, NSString *data, NSString *messageId) {
        // ...
    }];
```

### Display Unread Count

When other users send you a mark-as-read command, it is passed to ```readReceivedBlock:``` callback in ```[Jiver setEventHandlerConnectBlock:...]```. 

```objectivec
[Jiver setEventHandlerConnectBlock:^(JiverChannel *channel) {
        // ...
    } errorBlock:^(NSInteger code) {
        // ...
    } channelLeftBlock:^(JiverChannel *channel) {
        // ...
    } messageReceivedBlock:^(JiverMessage *message) {
        // ...
    } systemMessageReceivedBlock:^(JiverSystemMessage *message) {
        // ...
    } broadcastMessageReceivedBlock:^(JiverBroadcastMessage *message) {
        // ...
    } fileReceivedBlock:^(JiverFileLink *fileLink) {
        // ...
    } messagingStartedBlock:^(JiverMessagingChannel *channel) {
        // ...
    } messagingUpdatedBlock:^(JiverMessagingChannel *channel) {
        // ...
    } messagingEndedBlock:^(JiverMessagingChannel *channel) {
        // ...
    } allMessagingEndedBlock:^{
        // ...
    } messagingHiddenBlock:^(JiverMessagingChannel *channel) {
        // ...
    } allMessagingHiddenBlock:^{
        // ...
    } readReceivedBlock:^(JiverReadStatus *status) {
        [self setReadStatus:[[status user] guestId] andTimestamp:[status timestamp]];
        [self.messagingTableView reloadData];
    } typeStartReceivedBlock:^(JiverTypeStatus *status) {
        // ...
    } typeEndReceivedBlock:^(JiverTypeStatus *status) {
        // ...
    } allDataReceivedBlock:^(NSUInteger jiverDataType, int count) {
        // ...
    } messageDeliveryBlock:^(BOOL send, NSString *message, NSString *data, NSString *messageId) {
        // ...
    }];
```

If you received a mark-as-read command, update the read status of the current channel and reload the message table view.

```objectivec
- (void) setReadStatus:(NSString *)userId andTimestamp:(long long)ts
{
    if (readStatus == nil) {
        readStatus = [[NSMutableDictionary alloc] init];
    }
    
    if ([readStatus objectForKey:userId] == nil) {
        [readStatus setObject:[NSNumber numberWithLongLong:ts] forKey:userId];
    }
    else {
        long long oldTs = [[readStatus objectForKey:userId] longLongValue];
        if (oldTs < ts) {
            [readStatus setObject:[NSNumber numberWithLongLong:ts] forKey:userId];
        }
    }
}
```

When the current channel is updated by ```registerNotificationHandlerMessagingChannelUpdatedBlock:mentionUpdatedBlock:```, you have to update the read status of the channel. The following method is invoked in ```registerNotificationHandlerMessagingChannelUpdatedBlock:``` callback.

```objectivec
- (void) updateMessagingChannel:(JiverMessagingChannel *)channel
{
    [self.navigationBarTitle setTitle:[MyUtils generateMessagingTitle:currentChannel]];
    
    NSMutableDictionary *newReadStatus = [[NSMutableDictionary alloc] init];
    for (JiverMemberInMessagingChannel *member in [channel members]) {
        NSNumber *currentStatus = [readStatus objectForKey:[member guestId]];
        if (currentStatus == nil) {
            currentStatus = [NSNumber numberWithLongLong:0];
        }
        [newReadStatus setObject:[NSNumber numberWithLongLong:MAX([currentStatus longLongValue], [channel getLastReadMillis:[member guestId]])] forKey:[member guestId]];
    }
    
    if (readStatus == nil) {
        readStatus = [[NSMutableDictionary alloc] init];
    }
    [readStatus removeAllObjects];
    for (NSString *key in newReadStatus) {
        id value = [newReadStatus objectForKey:key];
        [readStatus setObject:value forKey:key];
    }
    [self.messagingTableView reloadData];
}
```

If you implement the read status for all messages, you can see the unread count for each of them.

![Typing Indicator](img/009_Screenshot.png)

## Implement Messaging List

Each user has a list of messaging channels which it joined. We will display the channel list on Messaging Tab.

Open ```MessagingChannelListViewController.m``` in Xcode.

![MessagingChannelListViewController.m](img/016_MessagingChannelListViewController_m.png)

The messaging channel list will be updated whenever each channel is updated. Each channel item in the list includes a title which consists of its members, the member count, the last message in the channel, the date of the last message and the unread message count.

```startJiver``` method is invoked when the messaging tab is selected and ```prepareCloseMessagingViewController``` of ```MessagingViewControllerDelegate``` and ```prepareCloseMessagingInviteSelectChannelViewController``` of ```MessagingInviteSelectChannelViewControllerDelegate``` are invoked. ```registerNotificationHandlerMessagingChannelUpdatedBlock``` callback returns an updated messaging channel, which should be used to update the corresponding channel.

```objectivec
- (void) startJiver
{
    [Jiver loginWithUserId:[Jiver deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    [Jiver registerNotificationHandlerMessagingChannelUpdatedBlock:^(JiverMessagingChannel *channel) {
        for (JiverMessagingChannel *oldChannel in channelArray) {
            if ([oldChannel getId] == [channel getId]) {
                [channelArray removeObject:oldChannel];
                break;
            }
        }
        [channelArray insertObject:channel atIndex:0];
        [self.messagingChannelListTableView reloadData];
    }
    mentionUpdatedBlock:^(JiverMention *mention) {

    }];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        messagingChannelListQuery = [Jiver queryMessagingChannelList];
        [messagingChannelListQuery setLimit:15];
        if ([messagingChannelListQuery hasNext]) {
            [messagingChannelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
                [channelArray removeAllObjects];
                [channelArray addObjectsFromArray:queryResult];
                [self.messagingChannelListTableView reloadData];
            } endBlock:^(NSInteger code) {
                
            }];
        }
        [Jiver joinChannel:@""];
        [Jiver connect];
    });
}
```

When the channel is clicked ```MessagingViewController``` has to be open.

```objectivec
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    JiverMessagingChannel *channel = (JiverMessagingChannel *)[channelArray objectAtIndex:[indexPath row]];
    
    UIStoryboard *storyboard = [self storyboard];
    MessagingViewController *vc = [storyboard instantiateViewControllerWithIdentifier:@"MessagingViewController"];
    [vc setMessagingChannel:channel];
    [vc setDelegate:self];
    [self presentViewController:vc animated:YES completion:nil];
}
```

### Load Next Channel List

To see the next channel list, implement ```loadNextChannelList``` method. This method will be invoked when the ```messagingChannelListTableView`` draws the last cell. 

```objectivec
- (void)loadNextChannelList
{
    if (isLoadingChannel) {
        return;
    }
    isLoadingChannel = YES;
    
    if ([messagingChannelListQuery hasNext]) {
        [messagingChannelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
            [channelArray addObjectsFromArray:queryResult];
            [self.messagingChannelListTableView reloadData];
            
            isLoadingChannel = NO;
        } endBlock:^(NSInteger code) {
            isLoadingChannel = NO;
        }];
    }
}
``` 

![Messaging Channel List](img/017_Screenshot.png)
