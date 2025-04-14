from aiogram import Bot, Dispatcher, types, executor
from aiogram.types import ReplyKeyboardMarkup, KeyboardButton
import logging
import random

# Замените на ваш токен бота Telegram
API_TOKEN = 'YOUR_BOT_TOKEN_HERE'

# Включаем логирование для отладки
logging.basicConfig(level=logging.INFO)

# Инициализация бота и диспетчера событий
bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# Словарь для хранения коэффициентов, введённых каждым пользователем
user_data = {}

# Создание клавиатуры с двумя кнопками
keyboard = ReplyKeyboardMarkup(resize_keyboard=True)
keyboard.add(KeyboardButton("Сгенерировать раунды"))
keyboard.add(KeyboardButton("Получить прогноз"))

# Обработка команды /start
@dp.message_handler(commands=['start'])
async def start_handler(message: types.Message):
    await message.answer(
        "Привет! Я бот-предсказатель для Lucky Jet 💣\n\n"
        "Нажми 'Сгенерировать раунды', чтобы получить случайные коэффициенты, и я скажу, когда стоит заходить!",
        reply_markup=keyboard
    )

# Генерация случайных коэффициентов вместо ручного ввода
@dp.message_handler(lambda message: message.text == "Сгенерировать раунды")
async def generate_coeffs(message: types.Message):
    uid = message.from_user.id
    # Генерация 10 случайных коэффициентов от 1.00 до 5.00 с двумя знаками после запятой
    generated = [round(random.uniform(1.0, 5.0), 2) for _ in range(10)]
    user_data[uid] = generated
    await message.answer(f"🎲 Сгенерированные коэффициенты: {', '.join(map(str, generated))}\nТеперь нажми 'Получить прогноз' 🧠")

# Обработка нажатия кнопки "Получить прогноз"
@dp.message_handler(lambda message: message.text == "Получить прогноз")
async def give_prediction(message: types.Message):
    uid = message.from_user.id
    coeffs = user_data.get(uid)  # Получаем сохранённые коэффициенты пользователя

    if not coeffs:
        await message.answer("Сначала сгенерируй коэффициенты через кнопку 'Сгенерировать раунды'.")
        return

    # Подсчитываем количество "низких" (< 2) и "высоких" (>= 2) коэффициентов
    low = sum(1 for c in coeffs if c < 2)
    high = sum(1 for c in coeffs if c >= 2)

    # Простая стратегия: если было много низких, есть вероятность высокого
    if low > high:
        chance = random.choice([
            "⚡️ Вероятен высокий множитель — попробуй x2 или выше!",
            "🔥 Сейчас может быть зелёный! Держи!"
        ])
    else:
        chance = random.choice([
            "😐 Не самый лучший момент, подожди пару раундов.",
            "🕵️‍♂️ Пока идёт зелёная серия, но будь осторожен."
        ])

    # Отправляем пользователю прогноз
    await message.answer(f"📊 Анализ последних раундов завершён:\n{chance}")

# Запуск бота
if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
