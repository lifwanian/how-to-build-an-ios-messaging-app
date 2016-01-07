!REDIRECT "https://sendbird.gitbooks.io/how-to-build-an-ios-messaging-app/content/en/implement_open_chat.html"

# Implement Open Chat

## Create Channels on Inteage Dashboard

You have to create channels in order to implement open chat in your application.

1. Sign in at [Inteage Dashboard](https://dashboard.inteage.com).
2. Select **APPLICATION** on the side menu and click on your app.
3. Click **CHANNELS** tab.
4. Click **CREATE CHANNEL** button.
5. Fill up the form to create the channel you want.
6. Click **CREATE** button.

 
## Initialize Inteage Framework

Open the sample project and **AppDelegate.m** in Xcode.

![AppDelegate.m](./img/001_AppDelegate_m.png)

Initialize Inteage framework in ```application:didFinishLaunchingWithOptions:```

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [Inteage initAppId:@"<YOUR_APP_ID>"];
    
    return YES;
}
```
 
## Implement Channel Loading

Open **OpenChatChannelListViewController.m** in Xcode.

![OpenChatChannelListViewController.m](./img/002_OpenChatChannelListViewController_m.png)

Add [InteageChannelListQuery](http://docs.inteage.com/ref/ios/en/Classes/InteageChannelListQuery.html) object for querying the channels.

```objectivec
@interface OpenChatChannelListViewController ()<UITableViewDataSource, UITableViewDelegate, UISearchBarDelegate> {
    NSMutableArray *channelArray;
    BOOL isLoadingChannel;
    InteageChannelListQuery *channelListQuery;
}
```

Insert the following code to log in and query the channels at the bottom of ```viewDidLoad``` method.

```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    
    isLoadingChannel = NO;
    
    [self.openChatChannelListTableView setContentInset:UIEdgeInsetsMake(108, 0, 48, 0)];
    [self.openChatChannelListTableView setDelegate:self];
    [self.openChatChannelListTableView setDataSource:self];
    
    [self.channelSearchBar setDelegate:self];
    
    channelArray = [[NSMutableArray alloc] init];
    
    [self.openChatChannelListLoadingIndicator setHidden:YES];

    [Inteage loginWithUserId:[Inteage deviceUniqueID] andUserName:[MyUtils getUserName] andUserImageUrl:[MyUtils getUserProfileImage] andAccessToken:@""];
    channelListQuery = [Inteage queryChannelList];
    [channelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
        for (InteageChannel *channel in queryResult) {
            [channelArray addObject:channel];
        }
        [self.openChatChannelListTableView reloadData];
    } endBlock:^(NSError *error) {

    }];
}
```

**User ID** is used to identify users in your application. This project uses device ID(IDFV) as **User ID**. If there isn’t the user which has the user ID, Inteage will generate one.

Channel queries support pagination. Let’s try it out.

The ```loadNextChannelList``` method will be invoked when the ```UITableView``` for channels draws the last row. If you scroll to the bottom of the ```UITableView```, the next page of channels will be fetched from Inteage server and attached at the bottom of the ```UITableView```.

Modify ```loadNextChannelList``` like this:

```objectivec
- (void)loadNextChannelList
{    
    if (![channelListQuery hasNext]) {
        return;
    }
    
    if (isLoadingChannel) {
        return;
    }
    isLoadingChannel = YES;
    
    [channelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
        for (InteageChannel *channel in queryResult) {
            [channelArray addObject:channel];
        }
        [self.openChatChannelListTableView reloadData];
        isLoadingChannel = NO;
    } endBlock:^(NSError *error) {
        
    }];
}
```

## Channel Search

Inteage supports channel search which is implemented at ```UISearchBar``` in the sample project.

Modify [searchBarSearchButtonClicked:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISearchBarDelegate_Protocol/index.html#//apple_ref/occ/intfm/UISearchBarDelegate/searchBarSearchButtonClicked:) to query with a keyword and [searchBarCancelButtonClicked:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UISearchBarDelegate_Protocol/index.html#//apple_ref/occ/intfm/UISearchBarDelegate/searchBarCancelButtonClicked:) to clear the result.

```objectivec
- (void)searchBarCancelButtonClicked:(UISearchBar *)searchBar
{
    [channelArray removeAllObjects];
    [searchBar setText:@""];
    channelListQuery = [Inteage queryChannelList];
    [channelListQuery setQuery:@""];
    [channelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
        for (InteageChannel *channel in queryResult) {
            [channelArray addObject:channel];
        }
        [self.openChatChannelListTableView reloadData];
    } endBlock:^(NSError *error) {
        
    }];
}

- (void)searchBarSearchButtonClicked:(UISearchBar *)searchBar
{
    [channelArray removeAllObjects];
    channelListQuery = [Inteage queryChannelList];
    [channelListQuery setQuery:[searchBar text]];
    [channelListQuery nextWithResultBlock:^(NSMutableArray *queryResult) {
        for (InteageChannel *channel in queryResult) {
            [channelArray addObject:channel];
        }
        [self.openChatChannelListTableView reloadData];
    } endBlock:^(NSError *error) {
        
    }];
}
```

If you run the sample project, you can see the channels you created on the Inteage Dashboard.

![Channel List](./img/003_Screenshot.png) ![Channel List Querying](./img/005_Screenshot.png)

## Transfer a Message in Open Chat

Open **OpenChatChattingViewController.m** in Xcode.

![Open OpenChatChattingViewController.m](img/004_OpenChatChattingViewController_m.png)

The user interface for the chat consists of ```UITableView``` for displaying messages, ```UIButton``` for sending an image, ```UITextField``` for entering a message, and ```UIButton``` for sending a message. 

Modify ```startChattingWithPreviousMessage:``` method. This method is invoked when the chat begins. If you want to get previous messages during initialization, pass ```YES``` as an argument. Otherwise, pass ```NO```.

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

There are many blocks in this code. See this [link](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/setEventHandlerConnectBlock:errorBlock:channelLeftBlock:messageReceivedBlock:systemMessageReceivedBlock:broadcastMessageReceivedBlock:fileReceivedBlock:messagingStartedBlock:messagingUpdatedBlock:messagingEndedBlock:allMessagingEndedBlock:messagingHiddenBlock:allMessagingHiddenBlock:readReceivedBlock:typeStartReceivedBlock:typeEndReceivedBlock:allDataReceivedBlock:messageDeliveryBlock:) for detail.

Insert the following code to start the open chat at the bottom of ```viewDidLoad``` method.

```objectivec
- (void)viewDidLoad {
    //...
    
    [self startChattingWithPreviousMessage:YES];
}
```

To send a message, modify ```sendMessage``` method. This method is invoked by clicking the “Send” button or pressing the return key. [Inteage sendMessage:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/sendMessage:) method sends ```message``` in real-time.

```objectivec
- (void) sendMessage
{
    NSString *message = [self.messageTextField text];
    if ([message length] > 0) {
        [self.messageTextField setText:@""];
        [Inteage sendMessage:message];
    }
    scrollLocked = NO;
}
```

If you click the **File** button, ```clickSendFileButton:``` method will be invoked to open [UIImagePickerController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImagePickerController_Class/). Since [UIImagePickerController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImagePickerController_Class/) is used to select an image to send, modify the following method. [Inteage uploadFile:type:hasSizeOfFile:withCustomField:uploadBlock:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/uploadFile:type:hasSizeOfFile:withCustomField:uploadBlock:) which uploads the ```imageFileData``` to Inteage server. This method returns [InteageFileInfo](http://docs.inteage.com/ref/ios/en/Classes/InteageFileInfo.html) object which can be sent through [Inteage sendFile:](http://docs.inteage.com/ref/ios/en/Classes/Inteage.html#//api/name/sendFile:).

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

## Load Previous Messages

To view previous messages in the channel, implement ```loadPreviousMessages``` method. This method will be invoked when ```openChatChattingTableView`` draws the first cell. 

```objectivec
- (void) loadPreviousMessages {
    if (isLoadingMessage) {
        return;
    }
    isLoadingMessage = YES;

    [self.prevMessageLoadingIndicator setHidden:NO];
    [self.prevMessageLoadingIndicator startAnimating];
    [[Inteage queryMessageListInChannel:[currentChannel url]] prevWithMessageTs:firstMessageTimestamp andLimit:50 resultBlock:^(NSMutableArray *queryResult) {
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
                [self.openChatChattingTableView reloadData];
                if ([newMessages count] > 0) {
                    [self.openChatChattingTableView scrollToRowAtIndexPath:[NSIndexPath indexPathForRow:([newMessages count] - 1) inSection:0] atScrollPosition:UITableViewScrollPositionTop animated:NO];
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

## Run Sample Project

Build the project, join a channel and send a message. You need two devices or one device and an iOS simulator for testing. If you don’t have an extra device, you can use **OPERATIONS** on **Inteage Dashboard** to chat using an iOS simulator.
 
![Run the project](img/006_Screenshot.png)