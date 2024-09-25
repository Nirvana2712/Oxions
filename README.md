import sqlite3
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Updater, CommandHandler, CallbackContext

# Initialiser la base de données
def init_db():
    conn = sqlite3.connect('star_system.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users
                 (user_id INTEGER PRIMARY KEY, stars INTEGER, cagnotte REAL)''')
    conn.commit()
    conn.close()

# Fonction pour acheter des étoiles
def buy_star(update: Update, context: CallbackContext):
    user_id = update.message.from_user.id
    conn = sqlite3.connect('star_system.db')
    c = conn.cursor()
    
    # Vérifier si l'utilisateur existe
    c.execute("SELECT stars, cagnotte FROM users WHERE user_id=?", (user_id,))
    user = c.fetchone()
    
    # Calcul du multiplicateur exponentiel
    if user:
        stars = user[0] + 1
        cagnotte = user[1] + (1.05 * stars)  # Augmentation exponentielle
        c.execute("UPDATE users SET stars=?, cagnotte=? WHERE user_id=?", (stars, cagnotte, user_id))
    else:
        stars = 1
        cagnotte = 1.05  # Début de la cagnotte exponentielle
        c.execute("INSERT INTO users (user_id, stars, cagnotte) VALUES (?, ?, ?)", (user_id, stars, cagnotte))
    
    conn.commit()
    conn.close()

    update.message.reply_text(f"Vous avez acheté 1 étoile. Cagnotte totale : {cagnotte}€")

# Initialiser le bot Telegram
def start(update: Update, context: CallbackContext):
    update.message.reply_text("Bienvenue dans le système d'étoiles ! Utilisez /buy_star pour acheter des étoiles.")

def main():
    updater = Updater("VOTRE_TOKEN_TELEGRAM", use_context=True)
    dispatcher = updater.dispatcher

    dispatcher.add_handler(CommandHandler("start", start))
    dispatcher.add_handler(CommandHandler("buy_star", buy_star))

    init_db()

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
# Partager un lien et récompenser le parrain
def share_link(update: Update, context: CallbackContext):
    inviter_id = update.message.from_user.id
    invitee_id = context.args[0]  # L'ID du nouveau membre qui achète des étoiles via le lien de partage
    
    conn = sqlite3.connect('star_system.db')
    c = conn.cursor()

    # Mise à jour du parrain (inviter) et du filleul (invitee)
    c.execute("SELECT stars FROM users WHERE user_id=?", (invitee_id,))
    invitee_stars = c.fetchone()[0]
    
    # Récompenser l'inviteur et mettre à jour la cagnotte
    c.execute("SELECT stars, cagnotte FROM users WHERE user_id=?", (inviter_id,))
    inviter = c.fetchone()
    
    if inviter:
        new_stars = inviter[0] + (invitee_stars * 0.5)  # 50% des étoiles du filleul
        new_cagnotte = inviter[1] + (invitee_stars * 0.1)  # Augmenter la cagnotte pour chaque partage
        c.execute("UPDATE users SET stars=?, cagnotte=? WHERE user_id=?", (new_stars, new_cagnotte, inviter_id))

    conn.commit()
    conn.close()

    update.message.reply_text(f"Partage réussi ! Vous avez reçu {invitee_stars * 0.5} étoiles supplémentaires !")
git add .
git commit -m "https://t.me/Theprodjekt_bot"
git push origin main
