# wiki: https://github.com/OpenIPC/wiki

# Установка универсальной прошивки OpenIPC
https://habr.com/ru/articles/688386/

в данный момент камера прошита через coupler (версия Light):
https://github.com/OpenIPC/coupler

# TODO:
нужно попробовать прошить Ultimete версию:
https://openipc.org/cameras/vendors/goke/socs/gk7205v300?mac=00-12-33-59-d2-82&cip=10.2.0.32&sip=10.2.0.1&net=eth&rom=nor16m&var=ultimate&sd=nosd


# created bot
@openipc_alexxlabs_bot
api_token: 6964335036:AAFGCS_-wdVU-bxs_8mYm48AC3V-ateK9bA
chat_id:   -4008458890

{"ok":true,"result":[
    {
        "update_id":213735787,
        "message":{
            "message_id":3,"from":{
                "id":940080535,
                "is_bot":false,
                "first_name":"\u0410\u043b\u0435\u043a\u0441\u0430\u043d\u0434\u0440",
                "last_name":"\u041a\u0430\u043b\u0438\u043d",
                "language_code":"ru"
            },
            "chat":{
                "id":-4008458890,
                "title":"alexxlabs",
                "type":"group",
                "all_members_are_administrators":true
            },
            "date":1703317584,
            "new_chat_participant":{
                "id":6964335036,
                "is_bot":true,
                "first_name":"openipc",
                "username":"openipc_alexxlabs_bot"
            },
            "new_chat_member":{
                "id":6964335036,
                "is_bot":true,
                "first_name":"openipc",
                "username":"openipc_alexxlabs_bot"
            },
            "new_chat_members":[
                {"id":6964335036,"is_bot":true,"first_name":"openipc","username":"openipc_alexxlabs_bot"}
            ]
        }
    },{
        "update_id":213735788,
        "my_chat_member":{
            "chat":{"id":-4008458890,"title":"alexxlabs","type":"group","all_members_are_administrators":true},
            "from":{
                "id":940080535,
                "is_bot":false,
                "first_name":"\u0410\u043b\u0435\u043a\u0441\u0430\u043d\u0434\u0440",
                "last_name":"\u041a\u0430\u043b\u0438\u043d",
                "language_code":"ru"
            },
            "date":1703317584,
            "old_chat_member":{
                "user":{
                    "id":6964335036,
                    "is_bot":true,
                    "first_name":"openipc",
                    "username":"openipc_alexxlabs_bot"
                },
                "status":"left"
            },
            "new_chat_member":{
                "user":{
                    "id":6964335036,
                    "is_bot":true,
                    "first_name":"openipc",
                    "username":"openipc_alexxlabs_bot"
                },
                "status":"member"
            }
        }
    }
]}
# ----------------------------------------------------------------------------------
python burn --chip hi3516ev200 --file=u-boot-gk7205v300-universal.bin -p COM2 --break 
&& C:\portable\putty\PUTTY.EXE -serial COM4 -sercfg 115200,8,n,1,N

https://github.com/OpenIPC/burn
https://github.com/OpenIPC/firmware/releases
