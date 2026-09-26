import tkinter as tk
from random import randint, choice


# =========================
# НАЛАШТУВАННЯ
# =========================

BG = "#0B1020"
PANEL = "#151D35"
WHITE = "#FFFFFF"
GRAY = "#9CA3AF"
RED = "#EF4444"
GREEN = "#22C55E"
BLUE = "#38BDF8"
PURPLE = "#8B5CF6"
YELLOW = "#FACC15"
ORANGE = "#F97316"


# =========================
# ДАНІ ГРИ
# =========================

players = []
bots = {}

current_player = 0
current_boss = 0

player_hp = 100
boss_hp = 100

battle_running = False


# Зброя
damage_price = {
    "🔫 Пістолет": 50,
    "💥 Дробовик": 100,
    "⚡ Лазер": 150,
    "🚀 Ракетниця": 200
}

# Захист
survive_price = {
    "🛡️ Щит": 60,
    "🔵 Енергетичний щит": 100,
    "🧱 Броня": 120,
    "💨 Ухилення": 150,
    "🧪 Резист": 80
}

# Підказки
hints = {
    "🔫 Пістолет": "Надійна базова зброя.",
    "💥 Дробовик": "Сильна атака на близькій дистанції.",
    "⚡ Лазер": "Потужний лазерний промінь.",
    "🚀 Ракетниця": "Дуже сильна зброя!",

    "🛡️ Щит": "🛡️ Зменшує урон",
    "🔵 Енергетичний щит": "🔵 Сильно зменшує урон",
    "🧱 Броня": "🔩 Сильна броня",
    "💨 Ухилення": "💨 25% шанс ухилитися",
    "🧪 Резист": "🧱 Зменшує отриману шкоду"
}


# Боси:
# ім'я, emoji, HP, мінімальний урон, максимальний урон
bosses = [
    ("Броньований Титан", "🤖", 120, 10, 20),
    ("Вогняний Дракон", "🐉", 160, 15, 25),
    ("Кібер-Демон", "👹", 200, 18, 30),
    ("Фінальний БОС", "💀", 250, 20, 35)
]


# =========================
# ВІКНО
# =========================

window = tk.Tk()
window.title("🤖 ROBOT BATTLE")
window.geometry("900x700")
window.resizable(False, False)
window.configure(bg=BG)


# =========================
# СЛУЖБОВІ ФУНКЦІЇ
# =========================

def clear():
    """Очищає всі елементи вікна."""
    for widget in window.winfo_children():
        widget.destroy()


def title_label(text, size=30, color=WHITE):
    label = tk.Label(
        window,
        text=text,
        font=("Arial", size, "bold"),
        fg=color,
        bg=BG
    )
    label.pack(pady=15)
    return label


def make_button(parent, text, command, bg_color=PURPLE, width=25):
    return tk.Button(
        parent,
        text=text,
        command=command,
        width=width,
        bg=bg_color,
        fg=WHITE,
        font=("Arial", 11, "bold"),
        relief="flat",
        cursor="hand2"
    )


# =========================
# ГОЛОВНЕ МЕНЮ
# =========================

def menu():
    global name_entry
    global money_entry
    global players_label
    global message

    clear()

    title_label("🤖 ROBOT BATTLE", 32)

    tk.Label(
        window,
        text="Створи робота та переможи босів!",
        font=("Arial", 14),
        fg=GRAY,
        bg=BG
    ).pack()

    panel = tk.Frame(
        window,
        bg=PANEL,
        padx=35,
        pady=25
    )
    panel.pack(pady=25)

    tk.Label(
        panel,
        text="👤 Ім'я робота",
        font=("Arial", 14),
        fg=WHITE,
        bg=PANEL
    ).pack()

    name_entry = tk.Entry(
        panel,
        font=("Arial", 14),
        width=25,
        bg="#202A48",
        fg=WHITE,
        insertbackground=WHITE,
        relief="flat"
    )
    name_entry.pack(pady=10)

    make_button(
        panel,
        "➕ ДОДАТИ ГРАВЦЯ",
        add_player,
        PURPLE
    ).pack(pady=5)

    players_label = tk.Label(
        panel,
        text="Гравців: немає",
        font=("Arial", 12),
        fg=GRAY,
        bg=PANEL
    )
    players_label.pack(pady=10)

    tk.Label(
        panel,
        text="🪙 Стартові монети",
        font=("Arial", 14),
        fg=WHITE,
        bg=PANEL
    ).pack(pady=(10, 5))

    money_entry = tk.Entry(
        panel,
        font=("Arial", 14),
        width=15,
        bg="#202A48",
        fg=WHITE,
        insertbackground=WHITE,
        relief="flat"
    )
    money_entry.insert(0, "300")
    money_entry.pack()

    message = tk.Label(
        panel,
        text="💡 Порада: купи хоча б одну зброю.",
        font=("Arial", 11),
        fg=YELLOW,
        bg=PANEL
    )
    message.pack(pady=15)

    make_button(
        panel,
        "▶ ПОЧАТИ ГРУ",
        start_game,
        GREEN
    ).pack()

    # Якщо гравці вже були додані
    if players:
        players_label.config(
            text="Гравці: " + ", ".join(players)
        )


# =========================
# ДОДАВАННЯ ГРАВЦЯ
# =========================

def add_player():
    name = name_entry.get().strip()

    if len(name) < 3:
        message.config(
            text="❌ Ім'я має містити мінімум 3 символи",
            fg=RED
        )
        return

    name = name.capitalize()

    if name in players:
        message.config(
            text="❌ Такий гравець вже є",
            fg=RED
        )
        return

    players.append(name)

    bots[name] = {
        "coins": 300,
        "items": []
    }

    name_entry.delete(0, tk.END)

    players_label.config(
        text="Гравці: " + ", ".join(players)
    )

    message.config(
        text="✅ Гравця додано!",
        fg=GREEN
    )


# =========================
# ПОЧАТОК ГРИ
# =========================

def start_game():
    global current_player

    if len(players) == 0:
        message.config(
            text="❌ Додай хоча б одного гравця",
            fg=RED
        )
        return

    try:
        money = int(money_entry.get())
    except ValueError:
        message.config(
            text="❌ Введи число монет",
            fg=RED
        )
        return

    if money < 150:
        money = 150

    for player in players:
        bots[player]["coins"] = money

    current_player = 0

    shop()


# =========================
# МАГАЗИН
# =========================

def shop():
    global coins_label
    global status
    global next_button

    clear()

    player = players[current_player]

    title_label("🔧 МАЙСТЕРНЯ", 28)

    tk.Label(
        window,
        text="🤖 " + player,
        font=("Arial", 18, "bold"),
        fg=BLUE,
        bg=BG
    ).pack()

    coins_label = tk.Label(
        window,
        text="🪙 " + str(bots[player]["coins"]),
        font=("Arial", 16, "bold"),
        fg=YELLOW,
        bg=BG
    )
    coins_label.pack(pady=5)

    tk.Label(
        window,
        text="💡 Натисни предмет, щоб купити його",
        font=("Arial", 11),
        fg=GRAY,
        bg=BG
    ).pack()

    main = tk.Frame(window, bg=BG)
    main.pack(
        fill="both",
        expand=True,
        padx=30,
        pady=15
    )

    weapons = tk.Frame(
        main,
        bg=PANEL,
        padx=15,
        pady=10
    )
    weapons.pack(
        side="left",
        fill="both",
        expand=True,
        padx=10
    )

    defense = tk.Frame(
        main,
        bg=PANEL,
        padx=15,
        pady=10
    )
    defense.pack(
        side="right",
        fill="both",
        expand=True,
        padx=10
    )

    tk.Label(
        weapons,
        text="🔫 ЗБРОЯ",
        font=("Arial", 17, "bold"),
        fg=RED,
        bg=PANEL
    ).pack(pady=5)

    tk.Label(
        defense,
        text="🛡️ ЗАХИСТ",
        font=("Arial", 17, "bold"),
        fg=BLUE,
        bg=PANEL
    ).pack(pady=5)

    status = tk.Label(
        window,
        text="",
        font=("Arial", 11),
        fg=YELLOW,
        bg=BG
    )
    status.pack()

    # ---------- ПОКУПКА ----------

    def buy(item, price, button):
        if item in bots[player]["items"]:
            status.config(
                text="❌ Цей предмет вже куплено",
                fg=RED
            )
            return

        if bots[player]["coins"] < price:
            status.config(
                text="❌ Недостатньо монет",
                fg=RED
            )
            return

        bots[player]["coins"] -= price
        bots[player]["items"].append(item)

        coins_label.config(
            text="🪙 " + str(bots[player]["coins"])
        )

        button.config(
            text="✅ " + item,
            state="disabled"
        )

        status.config(
            text="💡 " + hints[item],
            fg=GREEN
        )

    # ---------- ЗБРОЯ ----------

    for item, price in damage_price.items():

        btn = tk.Button(
            weapons,
            text=item + " — " + str(price) + " 🪙",
            width=28,
            bg="#7F1D1D",
            fg=WHITE,
            font=("Arial", 10, "bold"),
            relief="flat",
            cursor="hand2"
        )

        btn.config(
            command=lambda i=item, p=price, b=btn:
            buy(i, p, b)
        )

        btn.pack(pady=3)

    # ---------- ЗАХИСТ ----------

    for item, price in survive_price.items():

        btn = tk.Button(
            defense,
            text=item + " — " + str(price) + " 🪙",
            width=28,
            bg="#075985",
            fg=WHITE,
            font=("Arial", 10, "bold"),
            relief="flat",
            cursor="hand2"
        )

        btn.config(
            command=lambda i=item, p=price, b=btn:
            buy(i, p, b)
        )

        btn.pack(pady=3)

    next_button = make_button(
        window,
        "➡️ ГОТОВО — НАСТУПНИЙ ГРАВЕЦЬ",
        next_player,
        PURPLE,
        30
    )
    next_button.pack(pady=15)


# =========================
# НАСТУПНИЙ ГРАВЕЦЬ
# =========================

def next_player():
    global current_player

    current_player += 1

    if current_player >= len(players):
        battle()
    else:
        shop()


# =========================
# БИТВА
# =========================

def battle():
    global player_hp
    global boss_hp
    global battle_running

    clear()

    player = players[0]
    boss = bosses[current_boss]

    player_hp = 100
    boss_hp = boss[2]
    battle_running = True

    title_label(
        boss[0] + " " + boss[1],
        28,
        RED
    )

    tk.Label(
        window,
        text="БОС " + str(current_boss + 1) +
             " / " + str(len(bosses)),
        font=("Arial", 12),
        fg=GRAY,
        bg=BG
    ).pack()

    global boss_hp_label
    boss_hp_label = tk.Label(
        window,
        text="❤️ БОС: " + str(boss_hp),
        font=("Arial", 17, "bold"),
        fg=WHITE,
        bg=BG
    )
    boss_hp_label.pack()

    global boss_bar
    boss_bar = tk.Canvas(
        window,
        width=650,
        height=25,
        bg="#202A48",
        highlightthickness=0
    )
    boss_bar.pack(pady=8)

    global player_hp_label
    player_hp_label = tk.Label(
        window,
        text="🤖 " + player + " ❤️ 100",
        font=("Arial", 17, "bold"),
        fg=BLUE,
        bg=BG
    )
    player_hp_label.pack(pady=10)

    global animation_label
    animation_label = tk.Label(
        window,
        text="⚔️",
        font=("Arial", 30, "bold"),
        fg=YELLOW,
        bg=BG
    )
    animation_label.pack(pady=5)

    global log
    log = tk.Text(
        window,
        width=80,
        height=8,
        bg=PANEL,
        fg=WHITE,
        font=("Consolas", 11),
        relief="flat"
    )
    log.pack(pady=10)

    log.insert(
        tk.END,
        "⚔️ БИТВА ПОЧАЛАСЯ!\n"
    )

    global attack_button
    attack_button = tk.Button(
        window,
        text="⚔️ АТАКУВАТИ",
        command=player_attack,
        width=25,
        height=2,
        bg=RED,
        fg=WHITE,
        font=("Arial", 13, "bold"),
        relief="flat",
        cursor="hand2"
    )
    attack_button.pack(pady=8)

    tk.Label(
        window,
        text="💡 Порада: захист зменшує шкоду боса.",
        font=("Arial", 11),
        fg=YELLOW,
        bg=BG
    ).pack()

    update()


# =========================
# ОНОВЛЕННЯ ЕКРАНУ
# =========================

def update():
    boss = bosses[current_boss]
    player = players[0]

    boss_hp_label.config(
        text="❤️ БОС: " + str(max(0, boss_hp))
    )

    player_hp_label.config(
        text="🤖 " + player +
             " ❤️ " + str(max(0, player_hp))
    )

    boss_bar.delete("all")

    width = 650 * max(0, boss_hp) / boss[2]

    boss_bar.create_rectangle(
        0,
        0,
        width,
        25,
        fill=RED,
        outline=""
    )


# =========================
# АТАКА ГРАВЦЯ
# =========================

def player_attack():
    global boss_hp
    global battle_running

    if not battle_running:
        return

    player = players[0]

    weapons = []

    for item in bots[player]["items"]:
        if item in damage_price:
            weapons.append(item)

    if len(weapons) == 0:
        log.insert(
            tk.END,
            "❌ У тебе немає зброї!\n"
        )
        return

    attack_button.config(state="disabled")

    weapon = choice(weapons)

    damage = damage_price[weapon] // 5
    damage += randint(-4, 8)

    if damage < 5:
        damage = 5

    boss_hp -= damage

    animation_label.config(
        text="💥 BOOM!"
    )

    log.insert(
        tk.END,
        "🔫 " + weapon +
        " → -" + str(damage) + " HP\n"
    )

    update()

    if boss_hp <= 0:
        battle_running = False
        window.after(700, win)
    else:
        window.after(700, boss_attack)


# =========================
# АТАКА БОСА
# =========================

def boss_attack():
    global player_hp

    if not battle_running:
        return

    player = players[0]
    boss = bosses[current_boss]

    damage = randint(
        boss[3],
        boss[4]
    )

    items = bots[player]["items"]

    # Щит
    if "🛡️ Щит" in items:
        damage -= 5

    # Енергетичний щит
    if "🔵 Енергетичний щит" in items:
        damage -= 8

    # Броня
    if "🧱 Броня" in items:
        damage -= 7

    # Резист
    if "🧪 Резист" in items:
        damage -= 4

    # Ухилення
    if "💨 Ухилення" in items:
        if randint(1, 100) <= 25:
            damage = 0

            log.insert(
                tk.END,
                "💨 Ти ухилився від атаки!\n"
            )

    if damage < 0:
        damage = 0

    player_hp -= damage

    animation_label.config(
        text="⚡ ATTACK!"
    )

    log.insert(
        tk.END,
        "👹 БОС → -" + str(damage) + " HP\n"
    )

    update()

    if player_hp <= 0:
        window.after(700, lose)
    else:
        window.after(
            700,
            lambda: finish_turn()
        )


def finish_turn():
    animation_label.config(text="⚔️")
    attack_button.config(state="normal")


# =========================
# ПЕРЕМОГА
# =========================

def win():
    global current_boss
    global battle_running

    battle_running = False

    clear()

    title_label(
        "🏆 ПЕРЕМОГА!",
        40,
        GREEN
    )

    boss = bosses[current_boss]

    tk.Label(
        window,
        text=boss[0] + " " +
             boss[1] +
             " ПЕРЕМОЖЕНИЙ!",
        font=("Arial", 18, "bold"),
        fg=WHITE,
        bg=BG
    ).pack()

    if current_boss < len(bosses) - 1:

        def next_boss():
            global current_boss

            current_boss += 1
            battle()

        make_button(
            window,
            "👹 НАСТУПНИЙ БОС",
            next_boss,
            PURPLE
        ).pack(pady=30)

    else:

        tk.Label(
            window,
            text="👑 ТИ ПЕРЕМІГ УСІХ БОСІВ!",
            font=("Arial", 20, "bold"),
            fg=YELLOW,
            bg=BG
        ).pack(pady=20)

        make_button(
            window,
            "🔄 НОВА ГРА",
            restart,
            GREEN
        ).pack(pady=20)


# =========================
# ПОРАЗКА
# =========================

def lose():
    global battle_running

    battle_running = False

    clear()

    title_label(
        "💀 ПОРАЗКА",
        40,
        RED
    )

    tk.Label(
        window,
        text="Бос переміг твого робота.",
        font=("Arial", 18),
        fg=WHITE,
        bg=BG
    ).pack()

    make_button(
        window,
        "🔄 СПРОБУВАТИ ЗНОВУ",
        restart,
        PURPLE
    ).pack(pady=30)


# =========================
# НОВА ГРА
# =========================

def restart():
    global current_player
    global current_boss
    global player_hp
    global boss_hp
    global battle_running

    current_player = 0
    current_boss = 0
    player_hp = 100
    boss_hp = 100
    battle_running = False

    players.clear()
    bots.clear()

    menu()


# =========================
# ЗАПУСК
# =========================

menu()

window.mainloop()
