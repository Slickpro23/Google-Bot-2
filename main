# API_TOKEN = '7073415062:AAFbuyp-dL4da9BFPO1zaF3yc28tQy2A10Y'
import telebot
import datetime
import mysql.connector
from google.oauth2 import service_account
from googleapiclient.discovery import build
from telebot import types

# Подключение к базе данных MySQL
mydb = mysql.connector.connect(
    host="akueyoungep.beget.app",
    port="3306",
    user="mymoney",
    password="g0MSegH7sZT*",
    database="mymoney",
    autocommit=True
)
if mydb.is_connected():
    print("Успешное подключение к базе данных MySQL")
    cursor = mydb.cursor()
    # Команда SQL для создания таблицы "IDGOOGLE", если она не существует
    create_table_query_idgoogle = """
        CREATE TABLE IF NOT EXISTS IDGOOGLE (numb INT AUTO_INCREMENT PRIMARY KEY, Id_google TEXT, Id_telegram INT)"""
    # Команда SQL для создания таблицы "PAY", если она не существует
    create_table_query_pay = '''
        CREATE TABLE IF NOT EXISTS PAY (id INT AUTO_INCREMENT PRIMARY KEY, telegram VARCHAR(255), name VARCHAR(255), 
        info VARCHAR(255), sum FLOAT, data DATETIME)'''
    # Выполнение команд создания таблиц
    cursor.execute(create_table_query_idgoogle)
    cursor.execute(create_table_query_pay)
    mydb.commit()
    print("Таблицы успешно созданы или уже существуют.")

# Инициализация бота
bot = telebot.TeleBot('7073415062:AAFbuyp-dL4da9BFPO1zaF3yc28tQy2A10Y')
# Аутентификация Google Calendar API
creds = service_account.Credentials.from_service_account_file('callbot.json')
service = build('calendar', 'v3', credentials=creds)
if service:
    print("Успешное подключение к Google Сервису")
user_data = {}
# Проверка и переподключение к базе данных MySQL
def check_db_connection():
    global mydb
    try:
        mydb.ping(reconnect=True)
        print("Подключение к базе данных MySQL активно")
    except mysql.connector.Error as e:
        print("Потеряно соединение с базой данных MySQL. Повторное подключение...")
        mydb = mysql.connector.connect(
            host="akueyoungep.beget.app",
            port="3306",
            user="mymoney",
            password="g0MSegH7sZT*",
            database="mymoney",
            autocommit=True
        )
        print("Успешное повторное подключение к базе данных MySQL")
# Функция для записи данных в базу данных MySQL
def save_to_mysql(calendar_id, telegram_user_id):
    check_db_connection()
    cursor = mydb.cursor()
    try:
        sql = "INSERT INTO IDGOOGLE (Id_google, id_telegram) VALUES (%s, %s)"
        val = (calendar_id, telegram_user_id)
        cursor.execute(sql, val)
        print("Данные успешно записаны в базу данных MySQL")
    except mysql.connector.Error as error:
        print("Ошибка при записи в базу данных MySQL:", error)
    finally:
        cursor.close()

# Обработчик команды "start"
@bot.message_handler(commands=['start'])
def start(message, user_id=None):
    check_db_connection()
    chat_id = message.chat.id
    cursor = mydb.cursor()
    cursor.execute("SELECT * FROM IDGOOGLE WHERE id_telegram = %s", (chat_id,))
    result = cursor.fetchone()

    if result:
        # Отправка сообщения
        bot.send_message(chat_id=chat_id, text=(
        "Выберите команду из меню или нажмите здесь:\n"
        "/clients - 📓 КЛИЕНТЫ\n"
        "/week_events - 🗓 РАСПИСАНИЕ НА НЕДЕЛЮ\n"
        "/pay_menu - 💵 Оплаты\n"
        "/settings - Настройки"
    ))
    else:
        bot.send_message(chat_id, "Введите название будущего календаря:")
        bot.register_next_step_handler(message, request_calendar_email)
def request_calendar_email(message):
    chat_id = message.chat.id
    calendar_name = message.text
    bot.send_message(chat_id, "Введите вашу почту, чтобы получить доступ к создаваемому календарю:")
    user_data[chat_id] = {"calendar_name": calendar_name}
    bot.register_next_step_handler(message, create_calendar)
def create_calendar(message):
    chat_id = message.chat.id
    calendar_email = message.text
    calendar_name = user_data[chat_id]["calendar_name"] + " " + calendar_email
    # Указываем часовой пояс Москвы (GMT+03:00)
    calendar = {
        'summary': calendar_name,
        'timeZone': 'Europe/Moscow'
    }

    created_calendar = service.calendars().insert(body=calendar).execute()
    calendar_id = created_calendar["id"]
    # Установка прав доступа для пользователя
    rule = {
        'scope': {
            'type': 'user',
            'value': calendar_email
        },
        'role': 'owner'  # Установка пользователя в роль владельца
    }
    service.acl().insert(calendarId=calendar_id, body=rule).execute()
    save_to_mysql(calendar_id, chat_id)
    calendar_link = f'https://calendar.google.com/calendar/r?cid={calendar_id}'
    bot.send_message(chat_id,
                     f'Календарь создан! \n'
                     f'ID календаря: {calendar_id}\n\n'
                     f'ВАЖНО - Если приложение уже установлено то сначала удалите его а потом пройдите по ссылке ниже\n'
                     f'👉 {calendar_link}\n\n'
                     f'Пользователь с почтой \n{calendar_email}\n'
                     f'получил права доступа к календарю.\n\n'
                     f'Второй вариант подключения - перейдите в свою почту что указали боту\n'
                     f'На неё придет сообщение о получение доступа к календарю - нажмите кнопку "добавить календарь" в этом сообщении\n\n'
                     f'Если вы все сделали правильно, то в списке ваших календарей появится тот, что вы подключили.\n'
                     f'Скачайте приложение Гугл календарь\n')

    bot.send_message(chat_id, text=(
        "Выберите команду из меню или нажмите здесь:\n"
        "/clients - 📓 КЛИЕНТЫ\n"
        "/week_events - 🗓 РАСПИСАНИЕ НА НЕДЕЛЮ\n"
        "/pay_menu - 💵 Оплаты\n"
        "/settings - Настройки"
    ))
###################
@bot.callback_query_handler(func=lambda call: call.data == 'menu')
def menu_handler(call):
    bot.send_message(chat_id=call.message.chat.id, text=(
        "Выберите команду из меню или нажмите здесь:\n"
        "/clients - 📓 КЛИЕНТЫ\n"
        "/week_events - 🗓 РАСПИСАНИЕ НА НЕДЕЛЮ\n"
        "/pay_menu - 💵 Оплаты\n"
        "/settings - Настройки"
    ))
##### Функция для получения списка событий из календаря
# 25053ebce773d294cfdd4c20e90ff9edbcbf5c30aaa4f6ce7b753dc359d7a0fc@group.calendar.google.com
def get_events(user_id):
    # Подключение к базе данных MySQL
    mydb = mysql.connector.connect(
        host="akueyoungep.beget.app",
        port="3306",
        user="mymoney",
        password="g0MSegH7sZT*",
        database="mymoney",
        autocommit=True
    )
    if mydb.is_connected():
        print("Успешное подключение к базе данных MySQL")
    # Получение calendarId из таблицы IDGOOGLE
    cursor = mydb.cursor()
    select_query = "SELECT Id_google FROM IDGOOGLE WHERE Id_telegram = %s"
    cursor.execute(select_query, (user_id,))
    id_google_result = cursor.fetchone()
    mydb.commit()
    if id_google_result:
        calendar_id = id_google_result[0]
        today = datetime.datetime.now()
        four_months_ago = today - datetime.timedelta(days=6 * 30)  # 6 months = 4 * 30 days
        time_min = four_months_ago.isoformat() + 'Z'
        events_result = service.events().list(
            calendarId=calendar_id,
            timeMin=time_min,
            maxResults=2500,
            singleEvents=True,
            orderBy='startTime'
        ).execute()
        events = events_result.get('items', [])
        formatted_events = []
        for event in events:
            event_summary = event.get('summary', 'Без названия')
            event_start = event.get('start', {}).get('dateTime')
            event_description = event.get('description', '-')
            event_location = event.get('location', '-')
            event_info = {
                'Клиент': event_summary,
                'Дата': None,
                'Время': None,
                'ПТ': event_description,
                'Сумма': event_location
            }
            if event_start:
                event_datetime = datetime.datetime.strptime(event_start, "%Y-%m-%dT%H:%M:%S%z")
                event_info['Дата'] = event_datetime.strftime("%d.%m.%Y")
                event_info['Время'] = event_datetime.strftime("%H:%M")
            formatted_events.append(event_info)
        return formatted_events
    else:
        return []
    # Обработчик команды "/get_events"
@bot.message_handler(commands=['get_events'])
def send_events(message):
    # Get the user id
    user_id = message.from_user.id
    # Get the events for the user
    events = get_events(user_id)
    if len(events) == 0:
        bot.reply_to(message, "У вас нет запланированных событий.")
    else:
        reply_message = "Все ваши тренировки за 6 месяцев:\n\n"
        for event in events:
            event_message = f"🔹 {event['Клиент']}\n"
            event_message += f"📅 {event['Дата']}"
            event_message += f"  {event['Время']}"
            event_message += f"  {event['ПТ']}"
            event_message += f"  {event['Сумма']}\n\n"
            # Splitting the message into chunks if it is too long
            if len(reply_message + event_message) > 4096:
                bot.reply_to(message, reply_message)
                reply_message = ""
            reply_message += event_message
        bot.reply_to(message, reply_message)
#/week_events
def week_events(user_id):
    # Подключение к базе данных MySQL
    mydb = mysql.connector.connect(
        host="akueyoungep.beget.app",
        port="3306",
        user="mymoney",
        password="g0MSegH7sZT*",
        database="mymoney",
        autocommit=True
    )
    if mydb.is_connected():
        print("Успешное подключение к базе данных MySQL")
    # Получение calendarId из таблицы IDGOOGLE
    cursor = mydb.cursor()
    select_query = "SELECT Id_google FROM IDGOOGLE WHERE Id_telegram = %s"
    cursor.execute(select_query, (user_id,))
    id_google_result = cursor.fetchone()
    mydb.commit()
    if id_google_result:
        calendar_id = id_google_result[0]
        events_result = service.events().list(
            calendarId=calendar_id,
            timeMin='2023-04-04T00:00:00Z',
            maxResults=2500,
            singleEvents=True,
            orderBy='startTime'
        ).execute()
        events = events_result.get('items', [])
        formatted_events = []
        for event in events:
            event_summary = event.get('summary', 'Без названия')
            event_start = event.get('start', {}).get('dateTime')
            event_description = event.get('description', '-')
            event_location = event.get('location', '-')
            event_info = {
                'Клиент': event_summary,
                'Дата': None,
                'Время': None,
                'ПТ': event_description,
                'Сумма': event_location
            }
            if event_start:
                event_datetime = datetime.datetime.strptime(event_start, "%Y-%m-%dT%H:%M:%S%z")
                event_info['Дата'] = event_datetime.strftime("%d.%m.%Y")
                event_info['Время'] = event_datetime.strftime("%H:%M")
            formatted_events.append(event_info)
        return formatted_events
    else:
        return []
@bot.message_handler(commands=['week_events'])
def send_week_events(message):
    user_id = message.from_user.id
    # Получение events за текущую неделю
    today = datetime.date.today()
    start_of_week = today - datetime.timedelta(days=today.weekday())
    end_of_week = start_of_week + datetime.timedelta(days=6)
    events = week_events(user_id)
    events_this_week = [event for event in events if start_of_week <= datetime.datetime.strptime(event['Дата'], "%d.%m.%Y").date() <= end_of_week]
    if len(events_this_week) == 0:
        bot.reply_to(message, "У вас нет запланированных событий на текущей неделе.")
    else:
        # Create a dictionary to group events by days of the week
        events_by_day = {}
        for event in events_this_week:
            event_date = datetime.datetime.strptime(event['Дата'], "%d.%m.%Y").date()
            day_of_week = event_date.strftime('%A')
            if day_of_week not in events_by_day:
                events_by_day[day_of_week] = []
            events_by_day[day_of_week].append(event)
        # Prepare the reply message
        reply_message = "Текущая неделя:\n"
        for day, events in events_by_day.items():
            reply_message += f"\n      📝{day} - {events[0]['Дата']} \n"
            for event in events:
                event_message = f"🔹 {event['Клиент']}"
                event_message += f" - {event['Время']}\n"
                # Splitting the message into chunks if it is too long
                if len(reply_message + event_message) > 4096:
                    bot.reply_to(message, reply_message)
                    reply_message = ""
                reply_message += event_message
        # Add the count of events to the reply message
        reply_message += f"\nНа неделе запланировано: {sum(len(events) for events in events_by_day.values())} тренировок"

        # Создаем клавиатуру "Назад"
        back_keyboard = types.InlineKeyboardMarkup()
        menu_button = types.InlineKeyboardButton(text="Вернутся в Меню", callback_data="menu")
        back_keyboard.add(menu_button)

        # Отправляем сообщение с расписанием и клавиатурой "Назад"
        bot.reply_to(message, reply_message, reply_markup=back_keyboard)

# Обработчик команды "/clients"
@bot.message_handler(commands=['clients'])
def send_event_names(message):
    user_id = message.chat.id
    events = get_events(user_id)
    if len(events) == 0:
        bot.reply_to(message, "У вас нет запланированных событий.")
    else:
        sorted_events = sorted(events, key=lambda event: datetime.datetime.strptime(event['Дата'], '%d.%m.%Y'))
        event_names = list(set([event['Клиент'] for event in sorted_events]))
        event_names_message = "Список всех ваших клиентов за последние 6 месяцев📓\n\n"
        chunks = [event_names[i:i + 30] for i in range(0, len(event_names), 30)]
        for chunk in chunks:
            keyboard = types.InlineKeyboardMarkup()
            for event_name in chunk:
                button = types.InlineKeyboardButton(text=event_name, callback_data=event_name)
                keyboard.add(button)
            bot.send_message(message.chat.id, event_names_message, reply_markup=keyboard)
        last_entry_date = datetime.datetime.strptime(sorted_events[-1]['Дата'], '%d.%m.%Y')
        last_entry_message = f"Последняя запись была сделана {last_entry_date.strftime('%d.%m.%Y')}"
        bot.send_message(message.chat.id, last_entry_message)
@bot.callback_query_handler(func=lambda call: True)
def handle_callback_query(call):
    event_name = call.data
    user_id = call.from_user.id  # Предполагая, что user_id доступен в объекте call
    events = get_events(user_id)
    # Фильтруем мероприятия по выбранному названию
    selected_events = [event for event in events if event['Клиент'] == event_name]
    if len(selected_events) == 0:
        bot.send_message(call.message.chat.id, "Нет мероприятий с выбранным названием.")
    else:
        event_details_message = f"📘 {event_name}:\n"
        weekday_mapping = {
            'Monday': 'ПН',
            'Tuesday': 'ВТ',
            'Wednesday': 'СР',
            'Thursday': 'ЧТ',
            'Friday': 'ПТ',
            'Saturday': 'СБ',
            'Sunday': 'ВС'
        }
        passed_events_count = 0
        upcoming_events_count = 0
        for event in selected_events:
            date = event['Дата']
            formatted_date = datetime.datetime.strptime(date, '%d.%m.%Y').strftime('%d.%m.%Y')
            weekday = datetime.datetime.strptime(date, '%d.%m.%Y').strftime('%A')
            weekday_short = weekday_mapping.get(weekday, weekday)
            time = event['Время']
            status = event['ПТ']
            event_details_message += f"\n📅 {formatted_date}  {weekday_short}  {time}  {status}"
            current_date = datetime.datetime.now().date()
            event_date = datetime.datetime.strptime(date, '%d.%m.%Y').date()
            if event_date < current_date:
                passed_events_count += 1
            else:
                upcoming_events_count += 1
        # Split the event details message into chunks
        max_length = 4080
        while event_details_message:
            chunk = event_details_message[:max_length]
            event_details_message = event_details_message[max_length:]
            bot.send_message(call.message.chat.id, chunk)
        summary_message = f"✔️ За последние 6 месяцев 🗓\n" \
                          f"✔️ Всего записано: {len(selected_events)} тренировок(ки)\n" \
                          f"✔️ Прошло: {passed_events_count} \n(не учитывая сегодняшний день)\n" \
                          f"✔️ Запланировано: {upcoming_events_count}"

        # Создаем клавиатуру "Назад"
        back_keyboard = types.InlineKeyboardMarkup()
        menu_button = types.InlineKeyboardButton(text="Вернуться в Меню", callback_data="menu")
        back_keyboard.add(menu_button)
        bot.send_message(call.message.chat.id, summary_message, reply_markup=back_keyboard)


@bot.message_handler(commands=['pay_menu'])
def pay_menu(message):
    # Отправка сообщения
    bot.send_message(message.chat.id, "Выберите действие:\n\n"
                                        "/pay_mounth Оплаты за месяц 📆\n"
                                        "Здесь вы сможете добавить или удалить запись об оплате\n\n"
                                        "/pay_year Оплаты за год - Статистика за последние 12 месяцев по полученным оплатам\n\n"
                                        "Остальные команды доступны в меню\n\n"
    )

# Функция-обработчик команды "/pay"
@bot.message_handler(commands=['pay'])
def pay(message):
    try:
        # Запрос ввода пользователя для данных клиента
        bot.send_message(message.chat.id,
                         "Пожалуйста, введите полное имя клиента: \n▪️ пример - Вася пупкин\n\nДля отмены записи нажмите 👉🏻/cancel")
        bot.register_next_step_handler(message, process_client_name)
    except Exception as e:
        bot.send_message(message.chat.id,
                         "Произошла ошибка при отправке запроса в базу данных. Пожалуйста, попробуйте еще раз.")

def process_client_name(message):
    client_name = message.text
    if client_name == '/cancel':
        bot.send_message(message.chat.id, "Запись оплаты отменена.")
        return
    try:
        # Запрос ввода пользователя для количества занятий
        bot.send_message(message.chat.id,
                         "Пожалуйста, введите количество занятий: \n▪️ только число без символов\n\nДля отмены записи нажмите 👉🏻/cancel")
        bot.register_next_step_handler(message, process_session_count, client_name)
    except Exception as e:
        bot.send_message(message.chat.id,
                         "Произошла ошибка при отправке запроса в базу данных. Пожалуйста, попробуйте еще раз.")

def process_session_count(message, client_name):
    session_count = message.text
    if session_count == '/cancel':
        bot.send_message(message.chat.id, "Запись оплаты отменена.")
        return

    if not session_count.isdigit():
        bot.send_message(message.chat.id,
                         "Пожалуйста, введите число без символов \n\nДля отмены записи нажмите 👉🏻/cancel")
        bot.register_next_step_handler(message, process_session_count, client_name)
        return
    try:
        # Запрос ввода пользователя для суммы оплаты
        bot.send_message(message.chat.id,
                         "Пожалуйста, введите сумму оплаты: \n▪️ только число без символов\n\nДля отмены записи нажмите 👉🏻/cancel")
        bot.register_next_step_handler(message, process_payment_amount, client_name, session_count)
    except Exception as e:
        bot.send_message(message.chat.id,
                         "Произошла ошибка при отправке запроса в базу данных. Пожалуйста, попробуйте еще раз.")

def process_payment_amount(message, client_name, session_count):
    payment_amount = message.text
    if payment_amount == '/cancel':
        bot.send_message(message.chat.id, "Запись оплаты отменена.")
        return

    if not payment_amount.isdigit():
        bot.send_message(message.chat.id,
                         "Пожалуйста, введите число без символов \n\nДля отмены записи нажмите 👉🏻/cancel")
        bot.register_next_step_handler(message, process_payment_amount, client_name, session_count)
        return

    try:
        # Подключение к базе данных MySQL
        mydb = mysql.connector.connect(
            host="akueyoungep.beget.app",
            port="3306",
            user="mymoney",
            password="g0MSegH7sZT*",
            database="mymoney",
            autocommit=True
        )
        if mydb.is_connected():
            print("Успешное подключение к базе данных MySQL")
            mydb.cursor()
        # Вставка данных оплаты в базу данных
        with mydb.cursor() as cursor:
            insert_query = "INSERT INTO PAY (telegram, name, info, sum, data) VALUES (%s, %s, %s, %s, %s)"
            current_datetime = datetime.datetime.now()
            values = (message.from_user.id, client_name, session_count, payment_amount, current_datetime)
            cursor.execute(insert_query, values)
            mydb.commit()
        # Отправка сообщения
        bot.send_message(message.chat.id, text=(
            "Данные об оплате сохранены.\n\n"
            "Выберите действие в этом меню:\n\n"
                "/pay  Добавить 📘\n"
                "/del_pay  Удалить ❌\n"
                "/pay_mounth Оплаты за месяц 📆\n"
                "/pay_year Оплаты за год 📆\n\n"
            "Или выберите команду из меню или нажмите её здесь чтобы вернуться в меню выше:\n"
                "/clients - 📓 КЛИЕНТЫ\n"
                "/week_events - 🗓 РАСПИСАНИЕ НА НЕДЕЛЮ\n"
                "/pay_menu - 💵 Оплаты\n"
                "/settings - Настройки"
        ))
    except Exception as e:
        bot.send_message(message.chat.id,
                         "Произошла ошибка при выполнении операции с базой данных. Пожалуйста, попробуйте еще раз.")

# Функция-обработчик команды "/pay_mounth"
@bot.message_handler(commands=['pay_mounth'])
def pay_mounth(message):
    # Подключение к базе данных MySQL
    mydb = mysql.connector.connect(
        host="akueyoungep.beget.app",
        port="3306",
        user="mymoney",
        password="g0MSegH7sZT*",
        database="mymoney",
        autocommit=True
    )
    if mydb.is_connected():
        print("Успешное подключение к базе данных MySQL")
        mydb.cursor()

    # Получение текущего года и месяца
    current_year = datetime.datetime.now().year
    current_month = datetime.datetime.now().month

    # Формирование запроса для выборки оплат за текущий месяц, связанных с Telegram ID пользователя
    with mydb.cursor() as cursor:
        select_query = "SELECT * FROM PAY WHERE YEAR(data) = %s AND MONTH(data) = %s AND telegram = %s"
        values = (current_year, current_month, message.from_user.id)
        cursor.execute(select_query, values)
        result = cursor.fetchall()

    # Вычисление суммы всех оплат за текущий месяц
    total_sum = 0
    for row in result:
        total_sum += row[4]
    # Отправка результатов пользователю с клавиатурой
    response = "Оплаты за текущий месяц:\n"
    counter = 1
    for row in result:
        response += f"№: {row[0]}  {row[2]}\n"
        response += f"Сумма: {row[4]} Кол-во ПТ: {row[3]}\n"
        response += f"Дата: {row[5]}\n"
        response += "------------------------\n"
        counter += 1
    # Добавление суммы всех оплат за текущий месяц к ответу
    response += f"Сумма оплат за месяц: {total_sum} P"

    bot.send_message(message.chat.id, response)
    # Отправка сообщения
    bot.send_message(message.chat.id, text=(
        "/pay  Добавить оплату📘\n"
        "/del_pay  Удалить оплату❌\n"
        "/pay_year Посмотреть оплаты за год 📆\n\n"
        "Остальные команды доступны в меню\n\n"
    ))
    mydb.commit()
# Обработка ответа пользователя на кнопку "del_pay"
@bot.message_handler(commands=['del_pay'])
def handle_del_pay(message):
    bot.send_message(message.chat.id, "Введите номер записи, которую хотите удалить:")
    bot.register_next_step_handler(message, delete_payment)
# Функция для удаления записи из базы данных
def delete_payment(message):
    # Подключение к базе данных MySQL
    mydb = mysql.connector.connect(
        host="akueyoungep.beget.app",
        port="3306",
        user="mymoney",
        password="g0MSegH7sZT*",
        database="mymoney",
        autocommit=True
    )
    if mydb.is_connected():
        print("Успешное подключение к базе данных MySQL")
        mydb.cursor()

    payment_number = message.text

    with mydb.cursor() as cursor:
        # Проверка существования записи в базе данных с указанным номером
        select_query = "SELECT * FROM PAY WHERE id = %s AND telegram = %s"
        values = (payment_number, message.from_user.id)
        cursor.execute(select_query, values)
        result = cursor.fetchone()

        if result is None:
            bot.send_message(message.chat.id, f"Такой записи с номером {payment_number} не существует.")
        else:
            # Удаление записи из базы данных с указанным номером
            delete_query = "DELETE FROM PAY WHERE id = %s AND telegram = %s"
            cursor.execute(delete_query, values)
            bot.send_message(message.chat.id, f"Запись с номером {payment_number} успешно удалена!")
            # Отправка сообщения
            bot.send_message(message.chat.id, text=(
                "/pay_mounth Оплаты за месяц 📆\n"
                "/pay_year Оплаты за год 📆\n\n"
                "Остальные команды доступны в меню\n\n"
            ))

    mydb.commit()

# Функция-обработчик команды "/pay_year"
@bot.message_handler(commands=['pay_year'])
def pay_year(message):
    # Подключение к базе данных MySQL
    mydb = mysql.connector.connect(
        host="akueyoungep.beget.app",
        port="3306",
        user="mymoney",
        password="g0MSegH7sZT*",
        database="mymoney",
        autocommit=True
    )
    if mydb.is_connected():
        print("Успешное подключение к базе данных MySQL")
        mydb.cursor()

    # Получение текущего года и месяца
    current_year = datetime.datetime.now().year
    current_month = datetime.datetime.now().month

    # Создание словаря для хранения сумм оплат для каждого месяца
    monthly_sums = {}

    # Формирование запроса для выборки оплат за текущий год, связанных с Telegram ID пользователя
    with mydb.cursor() as cursor:
        select_query = "SELECT * FROM PAY WHERE YEAR(data) = %s AND telegram = %s"
        values = (current_year, message.from_user.id)
        cursor.execute(select_query, values)
        result = cursor.fetchall()

    # Вычисление сумм оплат для каждого месяца
    for row in result:
        month = row[5].month
        if month in monthly_sums:
            monthly_sums[month] += row[4]
        else:
            monthly_sums[month] = row[4]

    # Вычисление суммы оплат за текущий месяц
    current_month_sum = monthly_sums.get(current_month, 0)

    # Вычисление суммы оплат за весь год
    total_year_sum = sum(monthly_sums.values())

    # Формирование строки ответа с суммами оплат для каждого месяца
    response = f"Оплаты за год {current_year}:\n"
    for month in range(1, 13):
        month_name = datetime.date(current_year, month, 1).strftime('%B')
        total_sum = monthly_sums.get(month, 0)
        response += f"{month_name} - {total_sum} P\n"

    # Добавление суммы за текущий месяц и за весь год к строке ответа
    response += f"\nСумма оплат за текущий месяц: {current_month_sum} P"
    response += f"\nСумма оплат за весь год: {total_year_sum} P"

    bot.send_message(message.chat.id, response)
    # Отправка сообщения
    bot.send_message(message.chat.id, text=(
                "/pay_mounth Оплаты за месяц 📆\n"
                "/pay_year Оплаты за год 📆\n\n"
                "Остальные команды доступны в меню\n\n"
            ))
    mydb.commit()
bot.polling()

