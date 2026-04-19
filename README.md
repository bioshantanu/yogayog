# যোগাযোগ (Yogayog) — Full-Stack Social Platform

A complete social media + academic research platform with secure authentication, admin approval workflow, and IRINS academic profile system.

---

## 🏗️ Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | HTML5, CSS3 (custom vars), Vanilla JS (ES Modules) |
| **Backend API** | PHP 8.1+ (REST, no framework) |
| **Database** | MySQL 8.0+ |
| **Auth** | Token-based sessions (DB-stored, HttpOnly cookies) |
| **Passwords** | bcrypt (cost=12) |
| **Uploads** | PHP file handling, base64 or multipart |
| **Email** | PHPMailer (SMTP) |
| **Server** | Apache 2.4+ with mod_rewrite |

---

## 📁 Project Structure

```
yogayog/
├── .htaccess                  ← Apache rules, security headers, CORS
├── .gitignore
│
├── config/
│   ├── config.php             ← App/DB config (⚠️ never commit)
│   ├── database.php           ← PDO singleton connection class
│   └── helpers.php            ← CORS, JSON response, validators
│
├── api/
│   ├── auth_middleware.php    ← JWT-like session guard
│   ├── auth.php               ← register / login / logout / me
│   ├── admin.php              ← dashboard / approvals / team / members
│   ├── posts.php              ← CRUD / react / comment / save / share
│   ├── irins.php              ← Full IRINS academic profile CRUD
│   ├── messages.php           ← Conversations & real-time messages
│   ├── notifications.php      ← Notification system
│   └── users.php              ← Profile / search / friends
│
├── database/
│   └── schema.sql             ← Complete MySQL schema + seed data
│
├── frontend/
│   ├── index.html             ← Main SPA entry point
│   └── assets/
│       ├── css/
│       │   └── main.css       ← All styles (CSS custom properties)
│       └── js/
│           ├── api.js         ← API client (all backend calls)
│           ├── app.js         ← Main app logic
│           ├── auth.js        ← Login/register/captcha
│           ├── admin.js       ← Admin panel logic
│           └── irins.js       ← IRINS editor
│
├── uploads/                   ← User uploaded files (auto-created)
│   ├── avatars/
│   ├── posts/
│   └── covers/
│
├── logs/                      ← Error logs (auto-created)
│
└── scripts/
    └── setup.py               ← Automated installer (Python 3)
```

---

## 🚀 Installation

### Prerequisites
- PHP 8.1+ with extensions: `pdo_mysql`, `mbstring`, `json`, `openssl`, `fileinfo`
- MySQL 8.0+
- Apache 2.4+ with `mod_rewrite`
- Python 3.8+ (for automated setup)

### Option A — Automated Setup (Recommended)
```bash
cd yogayog/
python3 scripts/setup.py
```
The script will:
1. Check all prerequisites
2. Prompt for DB credentials and app URL
3. Create the database and import schema
4. Generate `config/config.php` with a secure random secret
5. Create directories and set permissions
6. Verify the installation

### Option B — Manual Setup
```bash
# 1. Create database
mysql -u root -p < database/schema.sql

# 2. Copy and edit config
cp config/config.php.example config/config.php
# Edit DB_HOST, DB_NAME, DB_USER, DB_PASS, APP_URL

# 3. Create required directories
mkdir -p uploads/{avatars,posts,covers,stories} logs cache

# 4. Set permissions
chmod -R 755 api config frontend
chmod -R 777 uploads logs cache
```

### Virtual Host (Apache)
```apache
<VirtualHost *:80>
    ServerName yogayog.local
    DocumentRoot /var/www/html/yogayog
    <Directory /var/www/html/yogayog>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

---

## 🔐 Authentication Flow

```
User fills registration form
         │
         ▼
POST /api/auth.php?action=register
         │
         ▼
Stored in registration_requests (status=pending)
         │
         ▼
Admin logs in → sees pending request
         │
         ├─ Each admin casts vote (approve/reject)
         │
         ├─ 2+ approvals → auto-create users record → notify user
         │
         └─ 1+ rejection → reject → notify user
                   │
                   ▼
         Approved user logs in
                   │
                   ▼
POST /api/auth.php?action=login
         │
         ▼
Token stored in user_sessions → returned to frontend
         │
         ▼
All subsequent requests: Authorization: Bearer <token>
```

---

## 📡 API Reference

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/auth.php?action=register` | Submit registration |
| POST | `/api/auth.php?action=login` | User login |
| POST | `/api/auth.php?action=admin_login` | Admin login |
| POST | `/api/auth.php?action=logout` | Logout |
| GET  | `/api/auth.php?action=me` | Current user info |

### Admin (requires admin token)
| Method | Endpoint | Description |
|---|---|---|
| GET  | `/api/admin.php?action=dashboard` | Stats + pending |
| GET  | `/api/admin.php?action=pending` | All pending requests |
| POST | `/api/admin.php?action=vote` | Cast approval vote |
| POST | `/api/admin.php?action=approve` | Quick approve (super_admin) |
| POST | `/api/admin.php?action=reject` | Reject request |
| GET  | `/api/admin.php?action=members` | List all members |
| POST | `/api/admin.php?action=suspend` | Suspend user |
| GET  | `/api/admin.php?action=team` | Admin team |
| GET  | `/api/admin.php?action=logs` | Activity log |

### Posts
| Method | Endpoint | Description |
|---|---|---|
| GET  | `/api/posts.php?action=feed` | News feed |
| POST | `/api/posts.php?action=create` | Create post |
| POST | `/api/posts.php?action=react` | Like/react |
| POST | `/api/posts.php?action=comment` | Add comment |
| POST | `/api/posts.php?action=save` | Save post |

### IRINS
| Method | Endpoint | Description |
|---|---|---|
| GET  | `/api/irins.php?action=full` | Full profile |
| POST | `/api/irins.php?action=save_profile` | Update bio/links |
| POST | `/api/irins.php?action=add_education` | Add education |
| PUT  | `/api/irins.php?action=update_education&id=N` | Update education |
| DELETE | `/api/irins.php?action=delete_education&id=N` | Delete |
| POST | `/api/irins.php?action=add_publication` | Add publication |
| POST | `/api/irins.php?action=add_skill` | Add skill |
| POST | `/api/irins.php?action=add_award` | Add award |
| POST | `/api/irins.php?action=add_language` | Add language |

### Messages
| Method | Endpoint | Description |
|---|---|---|
| GET  | `/api/messages.php?action=conversations` | All conversations |
| GET  | `/api/messages.php?action=history&conversation_id=N` | Chat history |
| POST | `/api/messages.php?action=send` | Send message |

---

## 🗄️ Database Schema Summary

```
users                   ← Approved members
registration_requests   ← Pending signup queue
approval_votes          ← Per-admin votes
admin_team              ← Admin accounts
user_sessions           ← Auth tokens
admin_sessions          ← Admin auth tokens
posts                   ← Social posts
post_reactions          ← Likes/reactions
comments                ← Post comments
messages                ← Chat messages
conversations           ← Conversation threads
notifications           ← In-app notifications
friend_requests         ← Friendship system
groups                  ← Community groups
group_members           ← Group membership
events                  ← Events
event_attendees         ← Event RSVPs
marketplace_listings    ← Marketplace items
irins_profiles          ← IRINS base profile
irins_education         ← Education records
irins_experience        ← Work experience
irins_publications      ← Academic publications
irins_skills            ← Skills with proficiency
irins_awards            ← Awards & honours
irins_languages         ← Language proficiency
activity_log            ← Admin audit trail
stories                 ← 24-hour stories
saved_posts             ← Saved post collection
post_shares             ← Share tracking
```

---

## 🔒 Security Features

- ✅ **bcrypt** password hashing (cost=12)
- ✅ **PDO prepared statements** — zero SQL injection risk
- ✅ **Token-based sessions** stored in DB (not JWT — revocable)
- ✅ **HttpOnly cookies** + Bearer token dual support
- ✅ **Rate-limitable** API (add Redis/APCu in production)
- ✅ **CORS whitelist** — only approved origins
- ✅ **File upload sanitization** — type + size checks, no PHP in uploads/
- ✅ **XSS prevention** — `htmlspecialchars()` on all input
- ✅ **Security headers** — CSP, X-Frame-Options, X-XSS-Protection
- ✅ **Directory indexing** disabled
- ✅ **Config directory** blocked from web access

---

## 📋 Demo Credentials

| Role | Email | Password |
|---|---|---|
| Approved User | user@yogayog.in | User@123 |
| Super Admin | admin@yogayog.in | Admin@123 |
| Team Member | member@yogayog.in | Member@123 |

---

## 📦 Third-Party Libraries (Optional)

Install via Composer for full email support:
```bash
composer require phpmailer/phpmailer
composer require vlucas/phpdotenv    # for .env file support
```

---

## 🔄 Upgrade to WebSockets (Optional)

For real-time messaging, replace polling with:
```bash
npm install -g socket.io
```
Add a Node.js socket server in `scripts/socket_server.js` and connect from the frontend.
