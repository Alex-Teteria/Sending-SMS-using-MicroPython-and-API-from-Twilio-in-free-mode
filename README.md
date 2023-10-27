# Sending-SMS-using-MicroPython-and-API-from-Twilio-in-free-mode
Free sending of SMS via the microcontroller when some sensors are triggered  

Вимоги: плата мікроконтролера типу ESP8266. Реєстрація на сайті [Twilio](https://www.twilio.com/)

## Зміст

1. [Огляд](./README.md#1-огляд)
2. [Використання API від Twilio](./README.md#2-Використання-API-від-Twilio)
3. [Приклад](./README.md#3-Приклад)
4. [Install](./README.md#4-install)
   
## 1. Огляд

[Twilio](https://www.twilio.com/) надає можливість пересилати повідомлення SMS використовуючи API. Реєстрація на сайті [Twilio](https://www.twilio.com/) та безкоштовне пробне (trial) використання його сервісів передбачає надання одного телефонного номера, через який буде здійснюватись обмін повідомленнями. При цьому trial-режим використання має ряд обмежень, одне з яких можливість відправляти SMS тільки на один номер +380ХХХХХХХХХ, який зазначається при реєстрації та попередньо перевіряється надсиланням на нього коду. Але ніщо не заважає зареєструвати необхідну кількість телефонів як окремих користувачів, відповідно на кожен з них необхідно мати e-mail, який також перевіряється при реєстрації. Використовуючи мікроконтролер, наприклад ESP8266, можна автоматизувати процес розсилання повідомлень SMS по усім необхідним мобільним телефонам. Як [приклад](./README.md#5-Приклад), реалізовано відправлення SMS-повідомлень на окремі телефони при спрацьовуванні датчиків.

## 2. Використання API від Twilio 
API для надсилання SMS функціонує як POST-запит з наступними параметрами:  
- url = "\https://api.twilio.com/2010-04-01/Accounts/\ ___sid___ /Messages.json"
- data = "To=***to_num***&From=***from_num***&Body=***message***"
- auth = "***sid***", "***token***"
- headers = {"Content-Type": "application/x-www-form-urlencoded"}

Тобто для формування запиту ми повинні мати наступні дані, які отримуєм при реєстрації:
- ***sid*** - Account SID
- ***token*** - Auth Token
- ***from_num*** - My Twilio phone number
- ***to_num*** - номер, який зазначено та перевірено при реєстрації та на який буде надсилатись SMS
- ***message*** - власне саме повідомлення, яке ми хочемо відправити

Використання API засобами MicroPython, як приклад:
```python
class SMS_bot:
    def __init__(self, senders):
        self.senders = senders # set of senders's tuple: (to_num, from_num, auth_token, account_sid)
        
    def send_sms(self, message):
        for to_num, from_num, token, sid in self.senders:
            self.send_one(to_num, from_num, message, token, sid)

    def send_one(self, to_num, from_num, message, token, sid):
        headers = {'Content-Type': 'application/x-www-form-urlencoded'}
        data = f'To={to_num}&From={from_num}&Body={message}'
        url = f'https://api.twilio.com/2010-04-01/Accounts/{sid}/Messages.json'
        print("Trying to send SMS with Twilio")
        response = urequests.post(url,
                             data=data,
                             auth=(sid, token),
                             headers=headers)
        if response.status_code == 201:
            print(f"SMS sent to {to_num}!")
        else:
            print(f'Error sending SMS to {to_num}: {response.text}')
        response.close()
```
## 3. 
## 4. Install
Copy the code to your project.
