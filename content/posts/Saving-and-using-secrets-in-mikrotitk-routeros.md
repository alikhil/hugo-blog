---
title: "Storing and using secrets in Mikrotik RouterOS"
date: 2025-03-30 16:01:25

cover:
    image: "https://en.meming.world/images/en/8/8e/All_Right_Then%2C_Keep_Your_Secrets.jpg"
    alt: All Right Then, Keep Your Secrets

tags:
- routeros
- mikrotik
- secrets
- tutorial
- telegram

categories:
- selfhosting
- networking

---

Recently I have replaced my stock ISP router with [Mikrotik Hex S](https://mikrotik.com/product/hex_s). I have been using it for a while and I am very happy with it. It is a very powerful device which can be programmed and automated with built-in scripting language.

When I started writing my first scripts I faced a problem: how to store and use secrets in my scripts. I have found a solution and I want to share it with you.

<!--more-->

## The problem

Let's say I want to write a script that will send me telegram notifications. To do so I need to store my telegram bot token and chat id. Since I keep my RouterOS configuration in a git repository, I don't want to hardcode my secrets in the script.


```
:global sendTelegramMessage do={
    :local botToken "1234567890:ABCDEFGHIJKLMN"
    :local chatId "10000000"
    :local message "$1"

    # telegram notification
    /tool fetch url="https://api.telegram.org/bot$botToken/sendMessage\?chat_id=$chatId&text=$message" keep-result=no
}
```

## The solution

RouterOS has low level feature `/ppp secret` which can be used to store secrets. However, it could be inconvenient and a bit messy to use it directly in scripts. Instead, I would like to have more high level API to store and use secrets.

And, I have one. There is post in [mikrotik forum](https://forum.mikrotik.com/viewtopic.php?p=916159#p916159) by user with nickname **Amm0**. Ammo has shared a script of global function which can be used to store and retrieve secrets like this:

```
$SECRET set mySecret password="mySecretPassword"
:put [$SECRET get mySecret]
```

![Terminal Screenshot](/images/posts/mirotik-secrets/terminal.png)

Now, I modify my script to use this function to keep it clean and secure:

```
:global sendTelegramMessage do={
    :local botToken
    :set botToken "$[$SECRET get TELEGRAM_TOKEN]"
    :local chatId "$[$SECRET get CHAT_ID]"
    :local message "$1"

    # telegram notification
    /tool fetch url="https://api.telegram.org/bot$botToken/sendMessage\?chat_id=$chatId&text=$message" keep-result=no
}
```

<details>
  <summary>global SECRET function source code</summary>

```
### $SECRET
#   get <name>
#   set <name> password=<password>
#   remove <name
#   print
:global SECRET
:set $SECRET do={
    :global SECRET

    # helpers
    :local fixprofile do={
        :if ([/ppp profile find name="null"]) do={:put "nothing"} else={
            /ppp profile add bridge-learning=no change-tcp-mss=no local-address=0.0.0.0 name="null" only-one=yes remote-address=0.0.0.0 session-timeout=1s use-compression=no use-encryption=no use-mpls=no use-upnp=no
        }
    }
    :local lppp [:len [/ppp secret find where name=$2]]
    :local checkexist do={
        :if (lppp=0) do={
            :error "\$SECRET: cannot find $2 in secret store"
        }
    }

    # $SECRET
    :if ([:typeof $1]!="str") do={
        :put "\$SECRET"
        :put "   uses /ppp/secrets to store stuff like REST apikeys, or other sensative data"
        :put "\t\$SECRET print - prints stored secret passwords"
        :put "\t\$SECRET get <name> - gets a stored secret"
        :put "\t\$SECRET set <name> password=\"YOUR_SECRET\" - sets a secret password"
        :put "\t\$SECRET remove <name> - removes a secret"
    }

    # $SECRET print
    :if ($1~"^pr") do={
        /ppp secret print where comment~"\\\$SECRET"
        :return [:nothing]
    }

    # $SECRET get
    :if ($1~"get") do={
        $checkexist
       :return [/ppp secret get $2 password]
    }

    # $SECRET set
    :if ($1~"set|add") do={
        :if ([:typeof $password]="str") do={} else={:error "\$SECRET: password= required"}
        :if (lppp=0) do={
            /ppp secret add name=$2 password=$password
        } else={
            /ppp secret set $2 password=$password
        }
        $fixprofile
        /ppp secret set $2 comment="used by \$SECRET"
        /ppp secret set $2 profile="null"
        /ppp secret set $2 service="async"
        :return [$SECRET get $2]
    }

    # $SECRET remove
    :if ($1~"rm|rem|del") do={
        $checkexist
        :return [/ppp secret remove $2]
    }
    :error "\$SECRET: bad command"
}
  ```

</details>

## Conclusion

The good thing about this approach is that secrets storing and retrieving mechanism encapsulated and can be easily changed in the future without changing the scripts. Also, it is easy to use and understand.

Keep your secrets safe and happy scripting!
