import telebot
from telebot import types
import wikipedia


wikipedia.set_lang('ru')

def search_wiki(query):
    return wikipedia.search(query)


def wiki_page(page_name):
    page = wikipedia.page(page_name)
    return page.title, page.summary, page.url



bot = telebot.TeleBot('...')


@bot.callback_query_handler(func=lambda call: call.data)
def answer(call):
    title, summery, url = wiki_page(call.data)
    bot.send_message(call.message.chat.id, text=title)
    bot.send_message(call.message.chat.id, text=summery)
    bot.send_message(call.message.chat.id, text=url)

@bot.message_handler(commands=['wiki'])
def duck(message):
    text = ' '.join(message.text.split(' ')[1:])
    results = search_wiki(text)
    markup = types.InlineKeyboardMarkup()
    for res in results:
        markup.add(types.InlineKeyboardButton(res, callback_data=res))
    bot.send_message(message.chat.id, text='Смотри, что я нашел', reply_markup=markup)


@bot.message_handler(content_types=['voice'])
def voice_message(message: types.Message):
    if(message.voice):
        bot.send_message(message.chat.id, "use command 'help'")

@bot.message_handler(content_types=['photo'])
def photo_message(message: types.Message):
    if(message.photo):
        bot.send_message(message.chat.id, "use command 'help'")

@bot.message_handler(content_types=['text'])
def get_help(message):
    if message.text == '/help':
        bot.send_message(message.chat.id, '<b>Available commands:</b> \n/start - intialize the bot \n/rus - select russian language (default) \n/eng - select eng - language \n <s>bot can take only text,btw</s>', parse_mode='html' )
    elif message.text.lower() == '/start':
        bot.send_message(message.chat.id, '<b>Чтобы ввести запрос в вики. Используй команду "/wiki"</b>', parse_mode='html')
    elif message.text.lower() == '/rus':
        wikipedia.set_lang('ru')
        bot.send_message(message.chat.id, 'Википедия изменена на русский язык!')
    elif message.text.lower() == '/eng':
        wikipedia.set_lang('eu')
        bot.send_message(message.chat.id, 'Changed wikipedia language to English!')


bot.polling(none_stop=True)
