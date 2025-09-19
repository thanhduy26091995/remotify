<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import { GetSettings } from "../../wailsjs/go/main/App"
import { values } from "../../wailsjs/go/models"

// Define props to receive payload data from parent tabs
const props = defineProps({
  apnsPayload: {
    type: String,
    default: ''
  },
  fcmPayload: {
    type: String,
    default: ''
  }
})

// Client Platform options
const clientPlatforms = [
  { value: 'ios-swift', label: 'iOS Swift' },
  { value: 'ios-objc', label: 'iOS Objective-C' },
  { value: 'android-kotlin', label: 'Android Kotlin' },
  { value: 'android-java', label: 'Android Java' },
  { value: 'flutter', label: 'Flutter' },
  { value: 'react-native', label: 'React Native' }
]

// Server Language options
const serverLanguages = [
  { value: 'nodejs', label: 'Node.js' },
  { value: 'python', label: 'Python' },
  { value: 'go', label: 'Go' },
  { value: 'java', label: 'Java' },
  { value: 'csharp', label: 'C#' },
  { value: 'php', label: 'PHP' }
]

// Reactive state
const selectedClientPlatform = ref('ios-swift')
const selectedServerLanguage = ref('nodejs')
const editPayloadHere = ref(false)
const customPayload = ref<any>({
  "aps": {
    "alert": {
      "title": "Hello",
      "body": "World"
    }
  }
})
const customPayloadText = ref(JSON.stringify(customPayload.value, null, 2))

const isDarkMode = ref(false)

// Load theme on mount
onMounted(() => {
  GetSettings().then(data => {
    let settings = JSON.parse(data)
    isDarkMode.value = settings.data.theme_mode === 'dark'
  })
})

// Generated client code
const clientCode = computed(() => {
  const platform = selectedClientPlatform.value
  let payload: any = customPayload.value
  
  // If not editing payload here, use the default template or parent payload
  if (!editPayloadHere.value) {
    try {
      // Use APNS payload if available, otherwise use default
      if (props.apnsPayload) {
        payload = JSON.parse(props.apnsPayload)
      } else {
        payload = JSON.parse(values.PayloadTemplate.DefaultAPNS)
      }
    } catch (e) {
      // Fallback to current payload if parsing fails
      console.warn('Failed to parse payload, using current payload')
    }
  }
  
  // Extract title and body safely
  const title = payload?.aps?.alert?.title || payload?.notification?.title || "Hello"
  const body = payload?.aps?.alert?.body || payload?.notification?.body || "World"
  
  switch (platform) {
    case 'ios-swift':
      return `let content = UNMutableNotificationContent()
content.title = "${title}"
content.body = "${body}"
content.sound = UNNotificationSound.default

let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
let request = UNNotificationRequest(identifier: "notification", content: content, trigger: trigger)

UNUserNotificationCenter.current().add(request) { error in
    if let error = error {
        print("Error: \\(error.localizedDescription)")
    }
}`
    
    case 'ios-objc':
      return `UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc] init];
content.title = @"${title}";
content.body = @"${body}";
content.sound = [UNNotificationSound defaultSound];

UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger 
    triggerWithTimeInterval:1 repeats:NO];
    
UNNotificationRequest *request = [UNNotificationRequest 
    requestWithIdentifier:@"notification" content:content trigger:trigger];

[[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request 
    withCompletionHandler:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"Error: %@", error.localizedDescription);
        }
    }];`
    
    case 'android-kotlin':
      return `val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

val notification = NotificationCompat.Builder(this, CHANNEL_ID)
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("${title}")
    .setContentText("${body}")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build()

notificationManager.notify(1, notification)`
    
    case 'android-java':
      return `NotificationManager notificationManager = 
    (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);

Notification notification = new NotificationCompat.Builder(this, CHANNEL_ID)
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("${title}")
    .setContentText("${body}")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build();

notificationManager.notify(1, notification);`
    
    case 'flutter':
      return `final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
    FlutterLocalNotificationsPlugin();

const AndroidNotificationDetails androidPlatformChannelSpecifics =
    AndroidNotificationDetails(
        'channel_id', 'channel_name',
        importance: Importance.max,
        priority: Priority.high);

const NotificationDetails platformChannelSpecifics =
    NotificationDetails(android: androidPlatformChannelSpecifics);

await flutterLocalNotificationsPlugin.show(
    0,
    '${title}',
    '${body}',
    platformChannelSpecifics);`
    
    case 'react-native':
      return `import PushNotification from 'react-native-push-notification';

PushNotification.localNotification({
    title: "${title}",
    message: "${body}",
    playSound: true,
    soundName: 'default',
});`
    
    default:
      return '// Select a platform to generate code'
  }
})

// Generated server code
const serverCode = computed(() => {
  const language = selectedServerLanguage.value
  let payload: any = customPayload.value
  
  // If not editing payload here, use the default template or parent payload
  if (!editPayloadHere.value) {
    try {
      // Use APNS payload if available, otherwise use default
      if (props.apnsPayload) {
        payload = JSON.parse(props.apnsPayload)
      } else {
        payload = JSON.parse(values.PayloadTemplate.DefaultAPNS)
      }
    } catch (e) {
      // Fallback to current payload if parsing fails
      console.warn('Failed to parse payload, using current payload')
    }
  }
  
  // Extract title and body safely
  const title = payload?.aps?.alert?.title || payload?.notification?.title || "Hello"
  const body = payload?.aps?.alert?.body || payload?.notification?.body || "World"
  
  switch (language) {
    case 'nodejs':
      return `const admin = require('firebase-admin');

const message = {
    token: "DEVICE_TOKEN",
    notification: {
        title: "${title}",
        body: "${body}"
    },
    apns: {
        payload: {
            aps: {
                alert: {
                    title: "${title}",
                    body: "${body}"
                }
            }
        }
    }
};

admin.messaging().send(message)
    .then((response) => {
        console.log('Successfully sent message:', response);
    })
    .catch((error) => {
        console.log('Error sending message:', error);
    });`
    
    case 'python':
      return `from firebase_admin import messaging
import firebase_admin

message = messaging.Message(
    token="DEVICE_TOKEN",
    notification=messaging.Notification(
        title="${title}",
        body="${body}"
    ),
    apns=messaging.APNSConfig(
        payload=messaging.APNSPayload(
            aps=messaging.Aps(
                alert=messaging.ApsAlert(
                    title="${title}",
                    body="${body}"
                )
            )
        )
    )
)

response = messaging.send(message)
print(f'Successfully sent message: {response}')`
    
    case 'go':
      return `package main

import (
    "context"
    "log"
    
    firebase "firebase.google.com/go"
    "firebase.google.com/go/messaging"
)

func sendMessage() {
    message := &messaging.Message{
        Token: "DEVICE_TOKEN",
        Notification: &messaging.Notification{
            Title: "${title}",
            Body:  "${body}",
        },
        APNS: &messaging.APNSConfig{
            Payload: &messaging.APNSPayload{
                Aps: &messaging.Aps{
                    Alert: &messaging.ApsAlert{
                        Title: "${title}",
                        Body:  "${body}",
                    },
                },
            },
        },
    }
    
    response, err := client.Send(ctx, message)
    if err != nil {
        log.Fatalln(err)
    }
    
    log.Printf("Successfully sent message: %s", response)
}`
    
    case 'java':
      return `import com.google.firebase.messaging.*;

Message message = Message.builder()
    .setToken("DEVICE_TOKEN")
    .setNotification(Notification.builder()
        .setTitle("${title}")
        .setBody("${body}")
        .build())
    .setApnsConfig(ApnsConfig.builder()
        .setAps(Aps.builder()
            .setAlert(ApsAlert.builder()
                .setTitle("${title}")
                .setBody("${body}")
                .build())
            .build())
        .build())
    .build();

String response = FirebaseMessaging.getInstance().send(message);
System.out.println("Successfully sent message: " + response);`
    
    case 'csharp':
      return `using FirebaseAdmin.Messaging;

var message = new Message()
{
    Token = "DEVICE_TOKEN",
    Notification = new Notification()
    {
        Title = "${title}",
        Body = "${body}"
    },
    Apns = new ApnsConfig()
    {
        Aps = new Aps()
        {
            Alert = new ApsAlert()
            {
                Title = "${title}",
                Body = "${body}"
            }
        }
    }
};

string response = await FirebaseMessaging.DefaultInstance.SendAsync(message);
Console.WriteLine($"Successfully sent message: {response}");`
    
    case 'php':
      return `<?php
require_once __DIR__.'/vendor/autoload.php';

use Kreait\\Firebase\\Factory;
use Kreait\\Firebase\\Messaging\\CloudMessage;
use Kreait\\Firebase\\Messaging\\Notification;

$factory = (new Factory)->withServiceAccount('path/to/service-account.json');
$messaging = $factory->createMessaging();

$message = CloudMessage::withTarget('token', 'DEVICE_TOKEN')
    ->withNotification(Notification::create('${title}', '${body}'));

$result = $messaging->send($message);
echo 'Successfully sent message: ' . $result;
?>`
    
    default:
      return '// Select a server language to generate code'
  }
})

// Copy to clipboard functions
const copyClientCode = async () => {
  try {
    await navigator.clipboard.writeText(clientCode.value)
    // Could add a toast notification here
  } catch (err) {
    console.error('Failed to copy client code: ', err)
  }
}

const copyServerCode = async () => {
  try {
    await navigator.clipboard.writeText(serverCode.value)
    // Could add a toast notification here
  } catch (err) {
    console.error('Failed to copy server code: ', err)
  }
}

const resetPayload = () => {
  customPayload.value = {
    "aps": {
      "alert": {
        "title": "Hello",
        "body": "World"
      }
    }
  }
  customPayloadText.value = JSON.stringify(customPayload.value, null, 2)
}

const updatePayloadFromText = () => {
  try {
    customPayload.value = JSON.parse(customPayloadText.value)
  } catch (e) {
    // Invalid JSON, keep the current payload
    console.warn('Invalid JSON in payload editor')
  }
}

const downloadZip = () => {
  // TODO: Implement zip download functionality
  console.log('Download ZIP functionality not implemented yet')
}

</script>

<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-12">
        <!-- Platform Selection -->
        <div class="row mb-3">
          <div class="col-md-6">
            <label for="clientPlatform" class="form-label">Client Platform:</label>
            <select id="clientPlatform" class="form-select" v-model="selectedClientPlatform">
              <option v-for="platform in clientPlatforms" :key="platform.value" :value="platform.value">
                {{ platform.label }}
              </option>
            </select>
          </div>
          <div class="col-md-6">
            <label for="serverLanguage" class="form-label">Server Language:</label>
            <select id="serverLanguage" class="form-select" v-model="selectedServerLanguage">
              <option v-for="language in serverLanguages" :key="language.value" :value="language.value">
                {{ language.label }}
              </option>
            </select>
          </div>
        </div>

        <!-- Edit Payload Toggle -->
        <div class="row mb-3">
          <div class="col-12">
            <div class="form-check">
              <input class="form-check-input" type="checkbox" id="editPayloadHere" v-model="editPayloadHere">
              <label class="form-check-label" for="editPayloadHere">
                Edit Payload Here
              </label>
            </div>
            <small class="text-muted">
              When "Edit Payload Here" is OFF → use payload from APNS/FCM tab<br>
              When ON → show inline JSON editor
            </small>
          </div>
        </div>

        <!-- Payload Editor (conditional) -->
        <div v-if="editPayloadHere" class="row mb-3">
          <div class="col-12">
            <label class="form-label">Payload Editor:</label>
            <div class="border rounded p-3" :class="isDarkMode ? 'bg-dark text-light' : 'bg-light'">
              <textarea 
                class="form-control" 
                rows="8" 
                v-model="customPayloadText" 
                @input="updatePayloadFromText"
                style="font-family: 'Courier New', monospace;"
                :class="isDarkMode ? 'bg-dark text-light border-secondary' : ''"
              ></textarea>
            </div>
          </div>
        </div>

        <!-- Client Code Section -->
        <div class="row mb-4">
          <div class="col-12">
            <div class="d-flex justify-content-between align-items-center mb-2">
              <h5 class="mb-0">Client Code ({{ clientPlatforms.find(p => p.value === selectedClientPlatform)?.label }}):</h5>
              <button class="btn btn-outline-primary btn-sm" @click="copyClientCode">
                <i class="bi bi-clipboard"></i> Copy Client Code
              </button>
            </div>
            <div class="border rounded p-3" :class="isDarkMode ? 'bg-dark text-light' : 'bg-light'">
              <pre class="mb-0"><code>{{ clientCode }}</code></pre>
            </div>
          </div>
        </div>

        <!-- Server Code Section -->
        <div class="row mb-4">
          <div class="col-12">
            <div class="d-flex justify-content-between align-items-center mb-2">
              <h5 class="mb-0">Server Code ({{ serverLanguages.find(l => l.value === selectedServerLanguage)?.label }}):</h5>
              <button class="btn btn-outline-primary btn-sm" @click="copyServerCode">
                <i class="bi bi-clipboard"></i> Copy Server Code
              </button>
            </div>
            <div class="border rounded p-3" :class="isDarkMode ? 'bg-dark text-light' : 'bg-light'">
              <pre class="mb-0"><code>{{ serverCode }}</code></pre>
            </div>
          </div>
        </div>

        <!-- Action Buttons -->
        <div class="row">
          <div class="col-12">
            <div class="d-flex gap-2">
              <button class="btn btn-secondary" @click="resetPayload">
                <i class="bi bi-arrow-clockwise"></i> Reset
              </button>
              <button class="btn btn-success" @click="downloadZip">
                <i class="bi bi-download"></i> Download as .zip
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-size: 0.9rem;
  max-height: 400px;
  overflow-y: auto;
}

code {
  font-family: 'Courier New', Monaco, monospace;
}

.form-select, .form-control {
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.btn {
  transition: all 0.15s ease-in-out;
}
</style>