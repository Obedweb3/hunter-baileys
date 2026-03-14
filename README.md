<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=32&pause=1000&color=25D366&center=true&vCenter=true&width=600&lines=Hunter-Baileys+v2.1.1;WhatsApp+Web+API+Library;Powered+by+OBED+TECH" alt="Hunter-Baileys" />

<h3>A powerful, full-featured WhatsApp Web API library for Node.js</h3>

[![NPM Version](https://img.shields.io/npm/v/hunter-baileys?color=25D366&label=npm&logo=npm)](https://www.npmjs.com/package/hunter-baileys)
[![NPM Downloads](https://img.shields.io/npm/dm/hunter-baileys?color=25D366&logo=npm)](https://www.npmjs.com/package/hunter-baileys)
[![License](https://img.shields.io/github/license/Obedweb3/hunter-baileys?color=25D366)](LICENSE)
[![Node.js](https://img.shields.io/badge/node-%3E%3D18.0.0-25D366?logo=node.js)](https://nodejs.org)
[![GitHub Stars](https://img.shields.io/github/stars/Obedweb3/hunter-baileys?color=25D366&logo=github)](https://github.com/Obedweb3/hunter-baileys/stargazers)

**Maintained by [OBED TECH](https://github.com/Obedweb3)**

</div>

---

## ⚠️ Disclaimer

> This project is **not affiliated, associated, authorized, endorsed by, or in any way officially connected** with WhatsApp or Meta Platforms Inc.
> Use at your own discretion. Do **not** spam people with this library. We strongly discourage stalkerware, bulk or automated unsolicited messaging.

---

## 📋 Table of Contents

- [Features](#-features)
- [Requirements](#-requirements)
- [Installation](#-installation)
- [Quick Start](#-quick-start)
- [Authentication](#-authentication)
  - [QR Code](#method-1--qr-code)
  - [Pairing Code](#method-2--pairing-code-no-qr)
- [Sending Messages](#-sending-messages)
  - [Text](#1-text-message)
  - [Image](#2-image-message)
  - [Video](#3-video-message)
  - [Audio / Voice Note](#4-audio--voice-note)
  - [Document](#5-document--file)
  - [Sticker](#6-sticker)
  - [Location](#7-location)
  - [Contact Card](#8-contact-card)
  - [Poll](#9-poll)
  - [Reaction](#10-reaction-emoji)
  - [Reply / Quote](#11-reply--quote-a-message)
  - [Forward](#12-forward-a-message)
- [Receiving Messages](#-receiving-messages)
- [Group Management](#-group-management)
- [Privacy Settings](#-privacy-settings)
- [Profile Management](#-profile-management)
- [WhatsApp Channels](#-whatsapp-channels-newsletter)
- [Presence & Typing](#-presence--typing-indicator)
- [Utilities](#-utilities)
- [Events Reference](#-events-reference)
- [License](#-license)

---

## ✨ Features

| Feature | Status |
|---------|--------|
| ✅ Multi-device support | Fully supported |
| ✅ QR Code authentication | Fully supported |
| ✅ Pairing Code (no QR) | Fully supported |
| ✅ Send/receive all message types | Fully supported |
| ✅ Group management (create, add, remove, promote) | Fully supported |
| ✅ Group status/story sending | Exclusive feature |
| ✅ WhatsApp Channels (Newsletter) | Fully supported |
| ✅ Privacy settings control | Fully supported |
| ✅ Profile picture & name management | Fully supported |
| ✅ Poll messages & vote decryption | Fully supported |
| ✅ LID addressing (personal & groups) | Fully supported |
| ✅ Media download & upload | Fully supported |
| ✅ Link preview generation | Fully supported |
| ✅ Disappearing messages | Fully supported |
| ✅ Block / unblock contacts | Fully supported |

---

## 📦 Requirements

- **Node.js** v18.0.0 or higher
- **npm** v9.0.0 or higher
- **Git** (required for installing dependencies)

---

## 💾 Installation

```bash
npm install hunter-baileys
```

Or using yarn:
```bash
yarn add hunter-baileys
```

**Import in your project:**
```javascript
// CommonJS (recommended)
const { default: makeWASocket, useMultiFileAuthState, Browsers, DisconnectReason } = require('hunter-baileys')

// ES Modules / TypeScript
import pkg from 'hunter-baileys'
const { default: makeWASocket, useMultiFileAuthState, Browsers, DisconnectReason } = pkg
```

---

## 🚀 Quick Start

Here is the simplest possible bot — connects to WhatsApp and replies to `!ping`:

```javascript
const { default: makeWASocket, useMultiFileAuthState, DisconnectReason, Browsers } = require('hunter-baileys')
const { Boom } = require('@hapi/boom')

async function startBot() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')

    const sock = makeWASocket({
        auth: state,
        browser: Browsers.ubuntu('MyBot'),
        printQRInTerminal: true,
    })

    // Save credentials whenever they update
    sock.ev.on('creds.update', saveCreds)

    // Handle connection
    sock.ev.on('connection.update', ({ connection, lastDisconnect }) => {
        if (connection === 'close') {
            const shouldReconnect = (lastDisconnect?.error instanceof Boom)
                ? lastDisconnect.error.output?.statusCode !== DisconnectReason.loggedOut
                : true
            if (shouldReconnect) startBot()
        } else if (connection === 'open') {
            console.log('✅ Bot connected!')
        }
    })

    // Handle messages
    sock.ev.on('messages.upsert', async ({ messages }) => {
        const msg = messages[0]
        if (!msg.message || msg.key.fromMe) return

        const from = msg.key.remoteJid
        const text = msg.message?.conversation || msg.message?.extendedTextMessage?.text || ''

        if (text === '!ping') {
            await sock.sendMessage(from, { text: '🏓 Pong!' })
        }
    })
}

startBot()
```

---

## 🔐 Authentication

### Method 1 — QR Code

Scan the QR code that appears in your terminal with WhatsApp.

```javascript
const { default: makeWASocket, useMultiFileAuthState } = require('hunter-baileys')
const qrcode = require('qrcode-terminal')

async function connect() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false, // we handle QR ourselves
    })

    sock.ev.on('creds.update', saveCreds)

    sock.ev.on('connection.update', ({ qr, connection }) => {
        if (qr) {
            console.log('Scan this QR code:\n')
            qrcode.generate(qr, { small: true })
        }
        if (connection === 'open') console.log('✅ Connected!')
    })
}

connect()
```

### Method 2 — Pairing Code (no QR)

Link your WhatsApp using a 8-digit code — no need to scan anything.

```javascript
const { default: makeWASocket, useMultiFileAuthState } = require('hunter-baileys')

async function connectWithPairingCode(phoneNumber) {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info')

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: false,
    })

    sock.ev.on('creds.update', saveCreds)

    sock.ev.on('connection.update', async ({ connection }) => {
        if (connection === 'open') console.log('✅ Connected!')
    })

    // Request pairing code after a short delay
    if (!sock.authState.creds.registered) {
        await new Promise(r => setTimeout(r, 3000))
        // Phone number with country code, no + or spaces
        const code = await sock.requestPairingCode(phoneNumber)
        console.log(`Your pairing code: ${code}`)
        // Enter this code in WhatsApp > Linked Devices > Link with phone number
    }
}

connectWithPairingCode('254712345678')
```

---

## 💬 Sending Messages

### 1. Text message

```javascript
await sock.sendMessage(jid, { text: 'Hello from hunter-baileys! 👋' })
```

### 2. Image message

```javascript
const fs = require('fs')

// From a file
await sock.sendMessage(jid, {
    image: fs.readFileSync('./image.jpg'),
    caption: 'Check out this image! 🖼️'
})

// From a URL
await sock.sendMessage(jid, {
    image: { url: 'https://example.com/image.jpg' },
    caption: 'Image from URL'
})
```

### 3. Video message

```javascript
await sock.sendMessage(jid, {
    video: fs.readFileSync('./video.mp4'),
    caption: 'Watch this! 🎬',
    gifPlayback: false // set true to send as GIF
})
```

### 4. Audio / Voice note

```javascript
// Regular audio file
await sock.sendMessage(jid, {
    audio: fs.readFileSync('./audio.mp3'),
    mimetype: 'audio/mp4'
})

// Voice note (shows as microphone bubble)
await sock.sendMessage(jid, {
    audio: fs.readFileSync('./voice.ogg'),
    mimetype: 'audio/ogg; codecs=opus',
    ptt: true // ptt = push to talk = voice note
})
```

### 5. Document / File

```javascript
await sock.sendMessage(jid, {
    document: fs.readFileSync('./file.pdf'),
    mimetype: 'application/pdf',
    fileName: 'my-document.pdf',
    caption: 'Here is your file 📄'
})
```

### 6. Sticker

```javascript
await sock.sendMessage(jid, {
    sticker: fs.readFileSync('./sticker.webp')
})
```

### 7. Location

```javascript
await sock.sendMessage(jid, {
    location: {
        degreesLatitude: -1.2921,   // Nairobi, Kenya
        degreesLongitude: 36.8219,
        name: 'Nairobi City',
        address: 'Nairobi, Kenya'
    }
})
```

### 8. Contact card

```javascript
const vcard = `BEGIN:VCARD\nVERSION:3.0\nFN:Obed Tech\nTEL;type=CELL;waid=254712345678:+254 712 345 678\nEND:VCARD`

await sock.sendMessage(jid, {
    contacts: {
        displayName: 'Obed Tech',
        contacts: [{ vcard }]
    }
})
```

### 9. Poll

```javascript
await sock.sendMessage(jid, {
    poll: {
        name: 'What is your favourite feature?',
        values: ['Pairing Code', 'Group Management', 'Media Support', 'Privacy Control'],
        selectableCount: 1 // how many options user can select
    }
})
```

### 10. Reaction (emoji)

```javascript
await sock.sendMessage(jid, {
    react: {
        text: '❤️',       // the emoji to react with — use '' to remove reaction
        key: msg.key      // the key of the message to react to
    }
})
```

### 11. Reply / Quote a message

```javascript
// The second argument is the quoted message
await sock.sendMessage(
    jid,
    { text: 'This is my reply!' },
    { quoted: msg }  // pass the original message here
)
```

### 12. Forward a message

```javascript
const { generateForwardMessageContent, generateWAMessageFromContent } = require('hunter-baileys')

const forwardContent = generateForwardMessageContent(msg, false)
const forwardMsg = generateWAMessageFromContent(jid, forwardContent, {
    userJid: sock.user.id
})
await sock.relayMessage(jid, forwardMsg.message, { messageId: forwardMsg.key.id })
```

---

## 📥 Receiving Messages

```javascript
sock.ev.on('messages.upsert', async ({ messages, type }) => {
    for (const msg of messages) {
        if (!msg.message) continue      // ignore empty messages
        if (msg.key.fromMe) continue    // ignore messages sent by the bot itself

        const from    = msg.key.remoteJid              // who sent it / which chat
        const isGroup = from.endsWith('@g.us')         // is it a group?
        const sender  = msg.key.participant || from    // actual sender in groups
        const pushName = msg.pushName || 'Unknown'     // contact display name

        // Get message text — handles all message types
        const body = msg.message?.conversation
            || msg.message?.extendedTextMessage?.text
            || msg.message?.imageMessage?.caption
            || msg.message?.videoMessage?.caption
            || msg.message?.documentMessage?.caption
            || ''

        console.log(`📩 From: ${pushName} (${from})`)
        console.log(`💬 Text: ${body}`)

        // Get message type
        const { getContentType } = require('hunter-baileys')
        const msgType = getContentType(msg.message)
        console.log(`📎 Type: ${msgType}`)

        // --- Example commands ---
        if (body === '!ping') {
            await sock.sendMessage(from, { text: '🏓 Pong!' }, { quoted: msg })
        }

        if (body === '!info') {
            await sock.sendMessage(from, {
                text: `*Hunter-Baileys Bot*\n\nVersion: v2.1.1\nOwner: OBED TECH\nLibrary: hunter-baileys`
            })
        }

        if (body === '!groups') {
            const groups = await sock.groupFetchAllParticipating()
            const list = Object.values(groups).map(g => `• ${g.subject}`).join('\n')
            await sock.sendMessage(from, { text: `*Groups I am in:*\n\n${list}` })
        }
    }
})
```

---

## 👥 Group Management

```javascript
// Create a new group
const group = await sock.groupCreate('My Group', [
    '254712345678@s.whatsapp.net',
    '254700000000@s.whatsapp.net'
])
console.log('Created group:', group.id)

// Add participants
await sock.groupParticipantsUpdate(groupJid, ['254712345678@s.whatsapp.net'], 'add')

// Remove participants
await sock.groupParticipantsUpdate(groupJid, ['254712345678@s.whatsapp.net'], 'remove')

// Promote to admin
await sock.groupParticipantsUpdate(groupJid, ['254712345678@s.whatsapp.net'], 'promote')

// Demote from admin
await sock.groupParticipantsUpdate(groupJid, ['254712345678@s.whatsapp.net'], 'demote')

// Change group name
await sock.groupUpdateSubject(groupJid, 'New Group Name')

// Change group description
await sock.groupUpdateDescription(groupJid, 'This is our group description')

// Lock group — only admins can send messages
await sock.groupSettingUpdate(groupJid, 'announcement')

// Unlock group — everyone can send messages
await sock.groupSettingUpdate(groupJid, 'not_announcement')

// Get group info
const metadata = await sock.groupMetadata(groupJid)
console.log('Group name:', metadata.subject)
console.log('Participants:', metadata.participants.length)

// Get group invite link
const code = await sock.groupInviteCode(groupJid)
console.log('Invite link: https://chat.whatsapp.com/' + code)

// Revoke invite link (creates a new one)
await sock.groupRevokeInvite(groupJid)

// Leave group
await sock.groupLeave(groupJid)

// Get all groups the bot is in
const allGroups = await sock.groupFetchAllParticipating()
console.log('Total groups:', Object.keys(allGroups).length)

// Enable disappearing messages (86400 = 24h, 604800 = 7 days, 7776000 = 90 days)
await sock.groupToggleEphemeral(groupJid, 86400)

// Send status/story to group (EXCLUSIVE hunter-baileys feature!)
await sock.giftedStatus(groupJid, {
    text: '🔥 Hello everyone! This is a group status!'
})
```

---

## 🔒 Privacy Settings

```javascript
// Last seen — 'all' | 'contacts' | 'contact_blacklist' | 'none'
await sock.updateLastSeenPrivacy('contacts')

// Online status visibility
await sock.updateOnlinePrivacy('match_last_seen')

// Profile picture — 'all' | 'contacts' | 'contact_blacklist' | 'none'
await sock.updateProfilePicturePrivacy('contacts')

// Status/Stories — 'all' | 'contacts' | 'contact_blacklist' | 'none'
await sock.updateStatusPrivacy('contacts')

// Read receipts (blue ticks) — 'all' | 'none'
await sock.updateReadReceiptsPrivacy('all')

// Who can add you to groups — 'all' | 'contacts' | 'contact_blacklist'
await sock.updateGroupsAddPrivacy('contacts')

// Who can call you — 'all' | 'contacts' | 'contact_blacklist' | 'none'
await sock.updateCallPrivacy('contacts')

// Default disappearing messages timer for new chats
await sock.updateDefaultDisappearingMode(86400) // 24 hours

// Block a contact
await sock.updateBlockStatus('254712345678@s.whatsapp.net', 'block')

// Unblock a contact
await sock.updateBlockStatus('254712345678@s.whatsapp.net', 'unblock')

// Get full blocklist
const blocklist = await sock.fetchBlocklist()
console.log('Blocked contacts:', blocklist)

// Get all privacy settings
const privacy = await sock.fetchPrivacySettings(true)
console.log(privacy)
```

---

## 👤 Profile Management

```javascript
const fs = require('fs')

// Change bot display name
await sock.updateProfileName('Hunter Bot 🤖')

// Change About/bio text
await sock.updateProfileStatus('Powered by OBED TECH 🚀')

// Change profile picture
const img = await sharp(fs.readFileSync('./photo.jpg'))
    .resize(640, 640)
    .toBuffer()
await sock.updateProfilePicture(sock.user.id, img)

// Remove profile picture
await sock.removeProfilePicture(sock.user.id)

// Get profile picture URL of any contact
const ppUrl = await sock.profilePictureUrl('254712345678@s.whatsapp.net', 'image')
console.log('Profile pic URL:', ppUrl)

// Get status/About of any contact
const status = await sock.fetchStatus('254712345678@s.whatsapp.net')
console.log('About:', status?.status)

// Check if a number is on WhatsApp
const [result] = await sock.onWhatsApp('254712345678')
console.log('On WhatsApp:', result?.exists)
console.log('JID:', result?.jid)
```

---

## 📰 WhatsApp Channels (Newsletter)

```javascript
// Create a new channel
const channel = await sock.newsletterCreate({
    name: 'OBED TECH Updates',
    description: 'Latest news and updates from OBED TECH'
})
console.log('Channel ID:', channel.id)

// Get channel metadata
const meta = await sock.newsletterMetadata('invite', 'channelInviteCode')
console.log('Subscribers:', meta.subscribers)

// Follow a channel
await sock.newsletterFollow(channelId)

// Unfollow a channel
await sock.newsletterUnfollow(channelId)

// Mute a channel
await sock.newsletterMute(channelId)

// Update channel name
await sock.newsletterUpdateName(channelId, 'New Channel Name')

// Update channel description
await sock.newsletterUpdateDescription(channelId, 'New description here')

// Fetch channel messages/posts
const messages = await sock.newsletterFetchMessages('updates', channelId, null, 20)

// React to a channel post
await sock.newsletterReactMessage(channelId, serverId, '🔥')

// Get subscriber count
const count = await sock.newsletterSubscribers(channelId)
console.log('Subscribers:', count)

// Delete channel
await sock.newsletterDelete(channelId)
```

---

## 🟢 Presence & Typing Indicator

```javascript
// Show "typing..." indicator
await sock.sendPresenceUpdate('composing', jid)

// Show "recording audio..." indicator
await sock.sendPresenceUpdate('recording', jid)

// Show as online
await sock.sendPresenceUpdate('available', jid)

// Show as offline
await sock.sendPresenceUpdate('unavailable', jid)

// Subscribe to presence updates of a contact
await sock.presenceSubscribe(jid)

// Listen for presence changes
sock.ev.on('presence.update', ({ id, presences }) => {
    console.log(`${id} is now:`, presences)
})

// Example: show typing before sending a message
async function sendWithTyping(jid, text) {
    await sock.sendPresenceUpdate('composing', jid)
    await new Promise(r => setTimeout(r, 1500)) // wait 1.5s
    await sock.sendPresenceUpdate('paused', jid)
    await sock.sendMessage(jid, { text })
}
```

---

## 🛠 Utilities

```javascript
const {
    isJidGroup,
    isJidUser,
    isJidNewsletter,
    jidNormalizedUser,
    getContentType,
    extractMessageContent,
    downloadMediaMessage,
    generateMessageID
} = require('hunter-baileys')

// Check if a JID is a group
if (isJidGroup(jid)) console.log('This is a group')

// Check if a JID is a regular user
if (isJidUser(jid)) console.log('This is a user')

// Check if a JID is a WhatsApp Channel
if (isJidNewsletter(jid)) console.log('This is a channel')

// Normalize a JID
const normalized = jidNormalizedUser('254712345678@s.whatsapp.net')

// Get the type of a message
const type = getContentType(msg.message)
// Returns: 'conversation' | 'imageMessage' | 'videoMessage' | 'audioMessage' | etc.

// Download media from any message
const buffer = await downloadMediaMessage(msg, 'buffer', {})
fs.writeFileSync('./downloaded.' + type, buffer)

// Generate a unique message ID
const id = generateMessageID()

// Mark messages as read (blue ticks)
await sock.readMessages([msg.key])

// Star a message
await sock.star(jid, [msg.key], true)

// Unstar a message
await sock.star(jid, [msg.key], false)

// Reject an incoming call
sock.ev.on('call', async (calls) => {
    for (const call of calls) {
        await sock.rejectCall(call.id, call.from)
    }
})
```

---

## 📡 Events Reference

| Event | Fires when... |
|-------|--------------|
| `connection.update` | Connection status changes — QR, connected, disconnected |
| `creds.update` | Session credentials are updated — always call `saveCreds()` |
| `messages.upsert` | New messages arrive |
| `messages.update` | A message is edited, deleted, or its status changes |
| `messages.reaction` | Someone reacts to a message |
| `message-receipt.update` | Delivery/read receipts update (ticks) |
| `groups.update` | Group info changes (name, description, settings) |
| `group-participants.update` | Someone joins, leaves, is added or removed |
| `presence.update` | A contact's online/typing status changes |
| `chats.update` | Chat metadata changes |
| `contacts.update` | Contact info changes |
| `call` | Incoming voice or video call |
| `labels.association` | Label added or removed from a chat |
| `blocklist.update` | Block list changes |

---

## 📁 Project Structure

```
hunter-baileys/
├── lib/
│   ├── Socket/
│   │   ├── socket.js          # Core WebSocket connection
│   │   ├── messages-send.js   # All message sending functions
│   │   ├── messages-recv.js   # Message receiving & decoding
│   │   ├── groups.js          # Group management
│   │   ├── chats.js           # Chat management
│   │   ├── business.js        # Business account features
│   │   ├── newsletter.js      # WhatsApp Channels
│   │   └── gcstatus.js        # Group status sending (exclusive)
│   ├── Utils/
│   │   ├── use-multi-file-auth-state.js  # Session management
│   │   ├── messages-media.js             # Media handling
│   │   ├── crypto.js                     # Encryption utilities
│   │   └── ...
│   ├── Types/                 # TypeScript type definitions
│   └── Defaults/              # Default configurations
├── WAProto/                   # WhatsApp protobuf definitions
├── package.json
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome! Feel free to:

1. Fork the repository
2. Create a new branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add my feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

Made with ❤️ by **OBED TECH**

[![GitHub](https://img.shields.io/badge/GitHub-Obedweb3-25D366?logo=github)](https://github.com/Obedweb3)

⭐ If this library helped you, please give it a star on GitHub!

</div>
