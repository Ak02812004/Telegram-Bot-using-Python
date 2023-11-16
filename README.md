# Telegram-Bot-using-Python
from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext, MessageHandler, Filters, InlineQueryHandler
from pymongo import MongoClient

# Connect to MongoDB
mongo_client = MongoClient("mongodb://localhost:27017/")  # Update the connection string
db = mongo_client["telegram_bot_db"]
users_collection = db["users"]

# Telegram Bot Token
TOKEN = "YOUR_TELEGRAM_BOT_TOKEN"  # Replace with your bot token

# Function to handle /start command
def start(update: Update, context: CallbackContext) -> None:
    user_id = update.effective_user.id
    user_data = {"user_id": user_id, "username": update.effective_user.username}
    
    # Check if the user already exists in the database
    if not users_collection.find_one({"user_id": user_id}):
        users_collection.insert_one(user_data)
        update.message.reply_text("Welcome! You are now registered.")

# Function to handle /info command
def info(update: Update, context: CallbackContext) -> None:
    user_id = update.effective_user.id
    
    # Retrieve user data from the database
    user_data = users_collection.find_one({"user_id": user_id})
    
    if user_data:
        update.message.reply_text(f"User ID: {user_data['user_id']}\nUsername: {user_data['username']}")
    else:
        update.message.reply_text("You are not registered. Send /start to register.")

# Function to handle text messages
def echo(update: Update, context: CallbackContext) -> None:
    update.message.reply_text(f"You said: {update.message.text}")

# Function to handle inline queries
def inline_query(update: Update, context: CallbackContext) -> None:
    query = update.inline_query.query
    results = [
        {
            "type": "article",
            "id": "1",
            "title": "Result",
            "input_message_content": {"message_text": f"You queried: {query}"},
        }
    ]
    update.inline_query.answer(results)

def main() -> None:
    updater = Updater(TOKEN)

    # Register handlers
    dp = updater.dispatcher
    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("info", info))
    dp.add_handler(MessageHandler(Filters.text & ~Filters.command, echo))
    dp.add_handler(InlineQueryHandler(inline_query))

    # Start the Bot
    updater.start_polling()

    # Run the bot until you send a signal to stop
    updater.idle()

if __name__ == "__main__":
    main()
