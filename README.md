# Sending-SMS-using-MicroPython-and-API-from-Twilio-in-free-mode
Free sending of SMS via the microcontroller when some sensors are triggered  

Вимоги: плата мікроконтролера типу ESP8266. Реєстрація на сайті [Twilio](https://www.twilio.com/)

## Зміст

1. [Огляд](./README.md#1-огляд)
2. [Використання API від Twilio](./README.md#2-Використання-API-від-Twilio)
3. [Приклад реалізації](./README.md#3-Приклад-реалізації)
4. [Install](./README.md#4-install)
   
## 1. Огляд

[Twilio](https://www.twilio.com/) надає можливість пересилати повідомлення SMS використовуючи API. Реєстрація на сайті [Twilio](https://www.twilio.com/) та безкоштовне (trial) використання його сервісів передбачає надання одного телефонного номера, через який буде здійснюватись обмін повідомленнями. При цьому trial-режим використання має ряд обмежень, одне з яких - можливість відправляти SMS тільки на один номер +380ХХХХХХХХХ, який зазначається при реєстрації та попередньо перевіряється надсиланням на нього коду. Але ніщо не заважає зареєструвати необхідну кількість телефонів як окремих користувачів, відповідно на кожен з них необхідно мати e-mail, який також перевіряється при реєстрації. Використовуючи мікроконтролер, наприклад ESP8266, можна автоматизувати процес розсилання повідомлень SMS по усім необхідним мобільним телефонам. Як [приклад](./README.md#3-Приклад-реалізації), реалізовано відправлення SMS-повідомлень на окремі телефони при спрацьовуванні датчиків.

## 2. Використання API від Twilio 
API для надсилання SMS функціонує як POST-запит з наступними параметрами:  
- url = "https: //api.twilio.com/2010-04-01/Accounts/___sid___/Messages.json"
- data = "To=***to_num***&From=***from_num***&Body=***message***"
- auth = "***sid***", "***token***"
- headers = {"Content-Type": "application/x-www-form-urlencoded"}

Тобто для формування запиту ми повинні мати наступні дані, які отримуєм при реєстрації:
- ***sid*** - Account SID
- ***token*** - Auth Token
- ***from_num*** - My Twilio phone number
- ***to_num*** - номер, який зазначено та перевірено при реєстрації та на який буде надсилатись SMS  

Ну і власне ***message*** - саме повідомлення, яке ми хочемо відправити.

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
## 3. Приклад реалізації
```python
# При спрацюванні сенсорів відправляється смс
# Повідомлення має вигляд:
# Час зміни стану сенсорів \n Назва сенсора: Назва стану в який сенсор перейшов
# Працездатність перевірено на платі ESP8266

import network, machine, urequests
from machine import Timer

def connect(ssid, passwd):
    '''варіант для ESP8266, для ESP32 не підходить, wlan.status() вертає інші значення!
    '''
    wlan.disconnect()
    wlan.connect(ssid, passwd)
    while wlan.status() == 1:
        pass
    return wlan.status()

def print_wlan_status(num_status):
    print(num_status)
    d_oled = {3: 'No access point', 2: 'Incorrect password', 4: 'other problems',
              5: 'Connection success', 0: 'No activity'}
    print(d_oled[num_status] if num_status in d_oled else 'ERROR!')
    if num_status == 5:
        print(wlan.ifconfig())

class Set_time:
    '''
    Підводить годинник реалного часу (rtc) до поточного часу сервера, який вказано
    в параметрі url. Для формування запиту використовує бібліотеку urequests
    '''
    def __init__(self, rtc, time_zone=2, url='http://time.in.ua/'):
        self.rtc = rtc # примірник об'єкту machine.RTC()
        self.time_zone = time_zone # UTC+2
        self.url = url
        self.d_month = {'Jan': 1, 'Feb':2, 'Mar': 3, 'Apr': 4, 'May': 5, 'Jun': 6,
                        'Jul': 7, 'Aug': 8, 'Sep': 9, 'Oct': 10, 'Nov': 11, 'Dec': 12}
        self.d_weekday = {'Mon,': 1, 'Tue,': 2, 'Wed,': 3, 'Thu,': 4, 'Fri,': 5, 'Sat,': 6, 'Sun,': 7}
                
    def set_rtc(self):
        try:
            r = urequests.head(self.url)
            weekday, day, month, year, current_time, mt = r.headers['Date'].split()
            hours, minutes, seconds = [int(x) for x in current_time.split(':')]
            print(int(year), self.d_month[month], int(day), self.d_weekday[weekday], int(hours), int(minutes), int(seconds))
# Заводимо годинник:   
            self.rtc.datetime((int(year), self.d_month[month], int(day), self.d_weekday[weekday], int(hours)+self.time_zone, int(minutes), int(seconds), 200))
            return True
        except:
            print('Error! Unable to create request instance')
            return False

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

def get_time():
    year, month, day, weekday, hours, minutes, seconds, subseconds = rtc.datetime()
    str_min = '0' + str(minutes)
    str_h = '0' + str(hours)
    str_sec = '0' + str(seconds)
    return str_h[-2:] +':'+ str_min[-2:] +':'+ str_sec[-2:]

def sensors_activation(*args):
    global current_sensors
    l_sensors = [name[0] if sensor.value() else name[1] for sensor, name in zip(t_sensors, status_names)]
    if current_sensors != l_sensors:
        d_out = {a: c for a, b, c in zip(sensor_names, current_sensors, l_sensors) if b != c}
        current_sensors = l_sensors.copy()
        print(d_out)
        message = ':\n' + get_time() + '\n'
        for k, v in d_out.items():
            message += k + ': ' + v + '\n'
        sms_bot = SMS_bot(senders)
        sms_bot.send_sms(message)

def debounce(*args):
    tim_debounce.init(mode=Timer.ONE_SHOT, period=time_debounce, callback=sensors_activation)
        
def sensor_irq():
    sensor_1.irq(trigger=machine.Pin.IRQ_RISING | machine.Pin.IRQ_FALLING, handler=debounce)
    sensor_2.irq(trigger=machine.Pin.IRQ_RISING | machine.Pin.IRQ_FALLING, handler=debounce)

if __name__ == '__main__':
# SSID and password for WiFi: 
    wlan_id = '########'
    wlan_pass = '############'
# enable station interface and connect to WiFi access point:
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
# Встановлюємо WiFi з'єднання:
    print_wlan_status(connect(wlan_id, wlan_pass))

# Ініціалізація сенсорів 
    tim_debounce = Timer(1)
    sensor_1 = machine.Pin(5, machine.Pin.IN, machine.Pin.PULL_UP)
    sensor_2 = machine.Pin(10, machine.Pin.IN, machine.Pin.PULL_UP)
    t_sensors = (sensor_1, sensor_2)
    sensor_names = ('Door', 'Window') # Тут вказуємо умовні назви сенсорів
    status_names = ('Open', 'Closed'), ('Open', 'Normal') # Тут вказуємо умовні назви станів сенсорів (коли "1", коли "0")
    current_sensors = [name[0] if sensor.value() else name[1] for sensor, name in zip(t_sensors, status_names)]
    time_debounce = 1000 # час брязкоту при спрацюванні сенсорів в мілісекундах

# Your SMS senders Settings:
    senders = {('to_num_1', 'from_num_1', 'token_1', 'sid_1'),
               ('to_num_2', 'from_num_2', 'token_2', 'sid_2')
                }
# Ініціалізація годинника реального часу, підведення його до UTC+2:
    rtc = machine.RTC()
    set_time = Set_time(rtc)
    set_time.set_rtc()
# Запускаємо слідкування за сенсорами, як зовнішні переривання:
    sensor_irq()
    while True:
        pass
```
## 4. Install
Copy the code to your project.
