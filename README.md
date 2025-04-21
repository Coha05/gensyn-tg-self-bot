# SET UP YOUR OWN ALERT WITH TELEGRAM BOT YOURSELF
If you install gensyn node so sure you already have nodejs installed that good

## 1. Get your user id

Use this bot to get your own ID: https://t.me/userinfobot
![image](https://github.com/user-attachments/assets/d3f71190-a460-4e9e-9797-71dbc5989ff8)

## 2. Create Bot Directory
```
mkdir swarm-log-bot && cd swarm-log-bot
```
## 3. Initialize and Install Packages
```
npm init -y
npm install node-telegram-bot-api node-cron dotenv
```
## 4. Create your own telegram bot
1. Go to https://t.me/BotFather
2. /newbot
3. Do as follow
4. Click to your bot and /start it
![image](https://github.com/user-attachments/assets/bf4a3b72-4422-4007-859e-949db72a3dca)

## 5. Create .env File
```
sudo nano .env
```
Paste it and replace with your own ID and bot token
```
BOT_TOKEN=your_bot_token_here
TELEGRAM_USER_ID=your_telegram_user_id_here
```

## 6. Create Bot Script swarmLoggerBot.js
```
sudo nano swarmLoggerBot.js
```
```
require('dotenv').config();
const TelegramBot = require('node-telegram-bot-api');
const { exec } = require('child_process');
const fs = require('fs');
const cron = require('node-cron');

const BOT_TOKEN = process.env.BOT_TOKEN;
const TELEGRAM_USER_ID = process.env.TELEGRAM_USER_ID;

const bot = new TelegramBot(BOT_TOKEN, { polling: true });

function escapeHtml(text) {
  return text
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;");
}

function getScreenLogs(callback) {
  exec(`screen -S swarm -X hardcopy -h /tmp/screen-swarm.log`, (err) => {
    if (err) return callback(err);
    fs.readFile('/tmp/screen-swarm.log', 'utf8', (err, data) => {
      if (err) return callback(err);
      const lastLines = data.trim().split('\n').slice(-10).join('\n'); 
      callback(null, escapeHtml(lastLines));
    });
  });
}

cron.schedule('*/15 * * * *', () => {
  getScreenLogs((err, logs) => {
    if (err) {
      bot.sendMessage(TELEGRAM_USER_ID, `Error reading logs: ${err.message}`);
    } else {
      bot.sendMessage(TELEGRAM_USER_ID, `<b>Swarm Log Update (Last 10 Lines):</b>\n\n<pre>${logs}</pre>`, {
        parse_mode: 'HTML'
      });
    }
  });
});

bot.onText(/\/start/, (msg) => {
  if (msg.from.id.toString() === TELEGRAM_USER_ID) {
    bot.sendMessage(msg.chat.id, `Swarm Log Bot is active. Logs will be sent every 15 minutes.`);
  } else {
    bot.sendMessage(msg.chat.id, `Access denied.`);
  }
});

bot.onText(/\/check/, (msg) => {
  if (msg.from.id.toString() === TELEGRAM_USER_ID) {
    getScreenLogs((err, logs) => {
      if (err) {
        bot.sendMessage(msg.chat.id, `Error reading logs: ${err.message}`);
      } else {
        bot.sendMessage(msg.chat.id, `<b>Manual Check Log (Last 10 Lines):</b>\n\n<pre>${logs}</pre>`, {
          parse_mode: 'HTML'
        });
      }
    });
  } else {
    bot.sendMessage(msg.chat.id, `Access denied.`);
  }
});
```
**Look your actual screen name and replace to above line 15**
## 7. Install PM2 globally
```
npm install -g pm2
```
## 8. Start bot and run it in background
```
pm2 start swarmLoggerBot.js --name swarm-log-bot
```

![image](https://github.com/user-attachments/assets/fb19a44b-0d32-42b9-a8b4-59351601e454)
