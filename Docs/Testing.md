# Testing

Common examples you can use to send [`Courier Inbox`](https://github.com/trycourier/courier-ios/blob/feature/inbox-docs/Docs/Inbox.md) messages and [`Push Notifications`](https://github.com/trycourier/courier-ios/blob/feature/inbox-docs/Docs/PushNotifications.md) to your users.

&emsp;

// Where to get auth key
// Do not put this in prod

https://app.courier.com/settings/api-keys

TODO Call out overrides with link

## Inbox Message

<table>
<tr>
<td width="500px" align="left">Swift</td>
<td width="500px" align="left">HTTP</td>
</tr>
<tr width="600px">
<td> 

```swift
try await Courier.shared.sendMessage(
    authKey: "YOUR_AUTH_KEY",
    userId: "example_user_id",
    title: "Hey there 👋",
    message: "Have a great day 😁",
    providers: [.inbox]
)
```

</td>
<td>

```bash
curl --request POST \
  --url https://api.courier.com/send \
  --header 'Authorization: Bearer YOUR_AUTH_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "message": {
        "to": {
            "user_id": "example_user_id"
        },
        "content": {
            "title": "Hey there 👋",
            "body": "Have a great day 😁"
        },
        "routing": {
            "method": "all",
            "channels": [
                "inbox"
            ]
        }
    }
}'
```

</td>
</tr>
</table>

### Result

<img width="894" alt="apns-push" src="https://user-images.githubusercontent.com/6370613/229195536-57ed3323-73cf-480a-89bf-d123063ff02a.png">

&emsp;

## Inbox Message with Actions

<table>
<tr>
<td width="500px" align="left">Swift</td>
<td width="500px" align="left">HTTP</td>
</tr>
<tr width="600px">
<td> 

```swift
try await Courier.shared.sendMessage(
    authKey: "YOUR_AUTH_KEY",
    userId: "example_user_id",
    title: "Hey there 👋",
    message: "Have a great day 😁",
    providers: [.inbox]
)
```

</td>
<td>

```bash
curl --request POST \
  --url https://api.courier.com/send \
  --header 'Authorization: Bearer YOUR_AUTH_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "message": {
        "to": {
            "user_id": "example_user_id"
        },
        "content": {
            "title": "Hey there 👋",
            "body": "Have a great day 😁"
        },
        "routing": {
            "method": "all",
            "channels": [
                "inbox"
            ]
        }
    }
}'
```

</td>
</tr>
</table>

### Result

<img width="894" alt="apns-push" src="https://user-images.githubusercontent.com/6370613/229195553-ebfd9d53-2a4e-48b6-8ea5-237cafd84ed2.png">

