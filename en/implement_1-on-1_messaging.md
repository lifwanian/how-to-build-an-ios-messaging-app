# Implement 1-on-1 Messaging

1-on-1 messaging needs two users. JIVER identifies each user with User ID. The sample project provides the feature for messaging which the user can choose another user for obtaining ID of the user in the open chat channel.

## Start messaging

You need opponent's user ID in order to start 1-on-1 messaging. Each message in open chat has the sender's user ID. We will use it for messaging.

Open **OpenChatChattingViewController.m** in Xcode.

![OpenChatChattingViewController.m](img/005_OpenChatChattingViewController_m.png)

Implement ```tableView:didSelectRowAtIndexPath:``` for getting the user ID of the message's sender.

```objectivec
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
    UIAlertController *messageSubMenu;
    UIAlertAction *messageAction;
    UIAlertAction *messageCancelAction;

    if ([[messages objectAtIndex:indexPath.row] isKindOfClass:[JiverMessage class]]) {
        JiverMessage *message = (JiverMessage *)[messages objectAtIndex:indexPath.row];
        
        if ([[[message sender] guestId] isEqualToString:[Jiver getUserId]]) {
            return;
        }
        
        NSString *actionTitle = [NSString stringWithFormat:@"Start messaging with %@", [message getSenderName]];
        messageSubMenu = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];
        messageAction = [UIAlertAction actionWithTitle:actionTitle style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            [Jiver startMessagingWithUserId:[[message sender] guestId]];
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

You can see UIAlertController when you click other's message in the open chat like this:

![UIAlertController for starting messaging](img/008_Screenshot.png)

```
When [Jiver startMessagingWithUserId:] is invoked and succeeded, the callback [messagingStartedBlock:](http://docs.jiver.co/ref/ios/en/Classes/Jiver.html#//api/name/setEventHandlerConnectBlock:errorBlock:channelLeftBlock:messageReceivedBlock:systemMessageReceivedBlock:broadcastMessageReceivedBlock:fileReceivedBlock:messagingStartedBlock:messagingUpdatedBlock:messagingEndedBlock:allMessagingEndedBlock:messagingHiddenBlock:allMessagingHiddenBlock:readReceivedBlock:typeStartReceivedBlock:typeEndReceivedBlock:allDataReceivedBlock:messageDeliveryBlock:) will be invoked as well. The callback returns [JiverMessagingChannel](http://docs.jiver.co/ref/ios/en/Classes/JiverMessagingChannel.html) object, then you and the opponent can join the channel.

 