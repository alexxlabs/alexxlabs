# Здесь фокусник за четыре шага покажет как без боли
# и чтения документации собрать бот-оповещатель для telegram.
Пройдите регистрацию в telegram
Получите идентификатор нового бота (далее ${telegramm_bot_token}):
обратитесь к боту @BotFather c требованием создать нового бота (команда /newbot)
Получите Идентификатор беседы с ботом (далее ${telegramm_chat_id}):
Откройте диалог со своим созданным ботом и напишите ему произвольное сообщение

Откройте в браузере ссылку, заменив ${telegramm_bot_token} на полученный идентификатор от @BotFather
Fetch bot updates and look for the chat id:
curl https://api.telegram.org/bot${telegramm_bot_token}/getUpdates| jq '.result[0].message.chat.id'
result->message->chat->id, это и есть <chat-id>

Откройте браузер и перейдите по ссылке, заменив ${telegramm_bot_token} и ${telegramm_chat_id} на свои данные
Send a message using the HTTP API: https://core.telegram.org/bots/api#sendmessage
curl -X POST \
   -H 'Content-Type: application/json' \
   -d '{"chat_id": "${telegramm_chat_id}", "text": "This is a test from curl", "disable_notification": true}' \
   https://api.telegram.org/bot${telegramm_bot_token}/sendMessage

Тем самым Вы получите сообщение от бота на все свои устройства с клиентом telegram.
Последний вызов можно использовать в bat-файле или консоли или раздать друзьям не имеющим telegram
и пишущим с умного утюга ( но только очень хорошим друзьям так как ${telegramm_bot_token}
конфиденциальная информация вообще-то).

# Telegram Bot - how to get a group chat id?
In order to get the group chat id, do as follows:
- Add the Telegram BOT to the group.
- Get the list of updates for your BOT:
    https://api.telegram.org/bot<YourBOTToken>/getUpdates
Ex:
    https://api.telegram.org/bot123456789:jbd78sadvbdy63d37gda37bd8/getUpdates
- Look for the "chat" object:
  {"update_id":8393,"message":{"message_id":3,"from":{"id":7474,"first_name":"AAA"},
  "chat":{"id":<group_ID>,"title":""},
  "date":25497,"new_chat_participant":{"id":71,"first_name":"NAME","username":"YOUR_BOT_NAME"}}}
This is a sample of the response when you add your BOT into a group.
Use the "id" of the "chat" object to send your messages.
(If you created the new group with the bot and you only get {"ok":true,"result":[]},
remove and add the bot again to the group)
Private chart only works in image argoprojlabs/argocd-notifications:v1.1.0 or above.
# --------------------------------------------------------------------------------
