# Sending-SMS-using-MicroPython-and-API-from-Twilio-in-free-mode
Free sending of SMS via the microcontroller when some sensors are triggered  

Вимоги: плата мікроконтролера типу ESP8266. Реєстрація на сайті [Twilio](https://www.twilio.com/)

## Зміст

1. [Огляд](./README.md#1-огляд)
2. [Obtaining a free SMTP service for sending email](./README.md#2-Obtaining-a-free-SMTP-service-for-sending-email)
3. [Install](./README.md#3-install)
4. [Objects used](./README.md#4-Objects-used)
5. [Приклад](./README.md#5-Приклад)
   
## 1. Огляд

[Twilio](https://www.twilio.com/) надає можливість пересилати повідомлення SMS використовуючи API. Реєстрація на сайті [Twilio](https://www.twilio.com/) та безкоштовне пробне (trial) використання його сервісів передбачає надання одного телефонного номера, через який буде здійснюватись обмін повідомленнями. При цьому trial-режим використання має ряд обмежень, одне з яких можливість відправляти SMS тільки на один номер +380ХХХХХХХХХ, який зазначається при реєстрації та попередньо перевіряється надсиланням на нього коду. Але ніщо не заважає зареєструвати необхідну кількість телефонів як окремих користувачів, відповідно на кожен з них необхідно мати e-mail, який також перевіряється при реєстрації. Використовуючи мікроконтролер, наприклад ESP8266, можна автоматизувати процес розсилання повідомлень SMS по усім необхідним мобільним телефонам. Як [приклад](./README.md#5-Приклад), реалізовано відправлення SMS-повідомлень на окремі телефони при спрацьовуванні датчиків.

## 2. Obtaining a free SMTP service for sending email
Before starting all further actions, it is advisable to create a new e-mail account from which the mailing will take place. If something goes wrong and you get banned, it won't be your primary email address :blush:  
This newly created account must be used when registering on the SMTP mailing service.  
After creating the sender email, the next step is to get the SMTP email server settings. I haven't been able to get them through Gmail yet.
Therefore, after a short wandering, the choice settled on the following services, taking into account the availability of a free option:  
[Sendgrid](https://sendgrid.com/). Its free use allows you to send 100 letters per day.  
[Brevo](https://www.brevo.com/). Its free use allows you to send 300 letters per day.  
All of them require verification during registration by sending an SMS message code to the phone number you specified. When registering, you should specify the new e-mail that was created for sending messages.

## 3. Install
Copy the code to your project.
