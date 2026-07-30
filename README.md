Cafe — online ordering site for a café / fast-food business

A Django web application where customers browse a menu, build an order with add-ons and sauces, and check out without creating an account. Every completed order is pushed to the staff Telegram chat within seconds, so the kitchen sees it without opening an admin panel.

Built for a real ordering workflow rather than as a catalog demo: guest checkout, per-item customization, and staff notifications are the three things a small food business actually needs on day one.

What it does

Menu and categories. Products are grouped into categories, each with a photo, description, price and stock count. Availability flips automatically when stock reaches zero.

Per-item customization. A product can be marked as having options, which opens a selector of extras and sauces, each with its own surcharge. Item price is recalculated from the base price plus everything the customer selected.

Guest cart. Anonymous visitors get a cart bound to their session key; registered users get a persistent cart created automatically on signup. No one is forced to register in order to order food.

Smart cart merging. Adding the same product twice increases the quantity only when the extras and sauces match exactly. A cheeseburger with extra cheese and a plain cheeseburger stay separate lines instead of collapsing into one — which is the bug most simple cart implementations ship with.

Checkout and staff notification. The order form collects recipient name, address, phone and an optional comment. On submit the order is persisted, the cart is cleared, and a formatted summary — items, add-ons, per-line totals and the grand total — is sent to every Telegram chat ID configured in the environment.

Admin panel. Django admin for managing products, categories, extras and sauces.

Tech stack
Layer	Choice
Backend	Django 6.0.1 (Python)
Database	PostgreSQL (psycopg2-binary)
Frontend	Django templates, vanilla JS, AOS scroll animations
Images	Pillow, ImageField uploads
Static files	WhiteNoise with compressed manifest storage
Notifications	Telegram Bot API
Config	python-dotenv, environment variables only
Tests	Django test suite — 27 tests


Project structure
myshop/            project configuration
  settings.py      env-driven config, no hardcoded secrets
  urls.py          root routing, media serving in DEBUG
store/             the application
  models.py        Category, Product, ProductImage, Extra, Sauce,
                   Cart, CartItem, Order, OrderItem, Payment
  views.py         catalog, cart operations, checkout, auth
  forms.py         signup with unique-email validation, order form
  signals.py       auto-create a cart when a user registers
  context_processors.py   categories and cart counter for every template
  utils.py         Telegram messaging helper
  templates/store/ 12 templates
  static/          css, js, images
  tests/           27 tests across models, views, forms, urls, admin, utils


  Running it locally

  git clone <repository-url>
cd cafe
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt

Create a .env file in the project root:

SECRET_KEY=your-django-secret-key
DEBUG=True
ALLOWED_HOSTS=127.0.0.1,localhost
CSRF_TRUSTED_ORIGINS=

DB_NAME=cafe
DB_USER=postgres
DB_PASSWORD=your-password
DB_HOST=localhost
DB_PORT=5432

TELEGRAM_BOT_TOKEN=token-from-@BotFather
CHAT_IDS=123456789,987654321

Then:
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver

Open http://127.0.0.1:8000/, add products through /admin/, and place a test order to verify the Telegram notification arrives.

Run the tests:
python manage.py test

Notes and known limitations

Being honest about the rough edges, because they are visible in the code anyway:

The home page selects featured products by hardcoded IDs. It should be a is_featured boolean on the model with an admin checkbox. This is the first thing I would change.
Payment model exists but is unused — payment_view is a stub. Orders are currently cash or card on delivery; online payment was out of scope.
Prices are in UZS (So'm) and parts of the interface are in Uzbek and Russian, since the project was built for a specific market. Making it multi-currency and translation-ready would require django.utils.translation and a currency field.
Telegram delivery is synchronous. If the Telegram API is slow, the customer waits. Moving it to a Celery task would fix that.
No automated order-status flow. Staff see the order in Telegram and in the admin; there is no "accepted / preparing / delivered" state machine yet.
Security

Configuration is loaded from environment variables only — the Django secret key, database credentials and the Telegram bot token are never committed. The repository contains no .env file, and DEBUG defaults are read from the environment rather than being switched on in code.
