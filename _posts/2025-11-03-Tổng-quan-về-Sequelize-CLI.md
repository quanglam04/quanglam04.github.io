---
title: "Tổng quan về Sequelize CLI"
date: 2025-11-03 01:17:00  +0700
categories: [CLI]
tags: [cli]
---


## **1. SEQUELIZE CLI LÀ GÌ?**

<p align="center">
  <img src="/assets/images/sequelize-cli/1.png" alt="Image title_1" />
</p>

```
Sequelize CLI = Git cho Database

- Quản lý thay đổi cấu trúc database theo thời gian
- Làm việc nhóm dễ dàng (sync database giữa các dev)
- Version control cho database structure
- Rollback khi có lỗi
```

---

## **2. CÀI ĐẶT**

```bash
# Cài đặt packages
npm install sequelize sequelize-cli mysql2

# Hoặc với PostgreSQL
npm install sequelize sequelize-cli pg pg-hstore

# Hoặc với SQLite
npm install sequelize sequelize-cli sqlite3
```

---

## **3. CẤU TRÚC THỨ MỤC**

```
your-project/
├── config/
│   └── config.json          ← Cấu hình database (dev/test/prod)
├── migrations/              ← File thay đổi database
│   ├── 20250423192935-create-tables.js
│   └── 20250423192936-seed-admin.js
├── models/                  ← Định nghĩa Model (ORM)
│   ├── index.js
│   ├── user.js
│   └── role.js
└── seeders/                 ← Dữ liệu mẫu (demo data)
    └── 20250423-demo-users.js
```

---

## **4. CÁC LỆNH CƠ BẢN**

### **A. Khởi tạo**
```bash
# Tạo cấu trúc folders (chạy 1 lần đầu dự án)
npx sequelize-cli init
```

### **B. Tạo Migration**
```bash
# Tạo file migration mới
npx sequelize-cli migration:generate --name tên-migration

# Ví dụ:
npx sequelize-cli migration:generate --name create-users-table
npx sequelize-cli migration:generate --name add-email-to-users
```

### **C. Chạy Migration**
```bash
# Chạy tất cả migrations chưa thực thi
npx sequelize-cli db:migrate

# Xem trạng thái migrations
npx sequelize-cli db:migrate:status
```

### **D. Rollback Migration**
```bash
# Rollback migration cuối cùng
npx sequelize-cli db:migrate:undo

# Rollback tất cả
npx sequelize-cli db:migrate:undo:all

# Rollback đến migration cụ thể
npx sequelize-cli db:migrate:undo:all --to XXXXXX-migration-name.js
```

### **E. Seeders (Dữ liệu mẫu)**
```bash
# Tạo seeder
npx sequelize-cli seed:generate --name demo-users

# Chạy tất cả seeders
npx sequelize-cli db:seed:all

# Rollback seeders
npx sequelize-cli db:seed:undo:all
```

---

## **5. CẤU TRÚC FILE MIGRATION**

```javascript
'use strict';

module.exports = {
  // ĐI LÊN: Thực hiện thay đổi
  async up(queryInterface, Sequelize) {
    // Viết code thay đổi database ở đây
    await queryInterface.createTable('users', { ... });
  },

  // ĐI XUỐNG: Hoàn tác thay đổi (ngược lại với up)
  async down(queryInterface, Sequelize) {
    // Viết code rollback ở đây
    await queryInterface.dropTable('users');
  }
};
```

---

## **6. CÁC THAO TÁC THƯỜNG DÙNG TRONG MIGRATION**

### **A. Tạo/Xóa bảng**
```javascript
// up(): Tạo bảng
await queryInterface.createTable('users', {
  id: {
    type: Sequelize.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  username: {
    type: Sequelize.STRING(50),
    allowNull: false,
    unique: true
  },
  email: Sequelize.STRING(100),
  createdAt: Sequelize.DATE,
  updatedAt: Sequelize.DATE
});

// down(): Xóa bảng
await queryInterface.dropTable('users');
```

### **B. Thêm/Xóa cột**
```javascript
// up(): Thêm cột
await queryInterface.addColumn('users', 'phone', {
  type: Sequelize.STRING(20),
  allowNull: true
});

// down(): Xóa cột
await queryInterface.removeColumn('users', 'phone');
```

### **C. Thay đổi cột**
```javascript
// up(): Sửa cột
await queryInterface.changeColumn('users', 'email', {
  type: Sequelize.STRING(255),
  allowNull: false
});

// down(): Đổi lại
await queryInterface.changeColumn('users', 'email', {
  type: Sequelize.STRING(100),
  allowNull: true
});
```

### **D. Đổi tên cột**
```javascript
// up(): Đổi tên cột
await queryInterface.renameColumn('users', 'username', 'user_name');

// down(): Đổi lại
await queryInterface.renameColumn('users', 'user_name', 'username');
```

### **E. Thêm/Xóa index**
```javascript
// up(): Tạo index
await queryInterface.addIndex('users', ['email']);
await queryInterface.addIndex('users', ['username', 'email']); // Composite

// down(): Xóa index
await queryInterface.removeIndex('users', ['email']);
```

### **F. Thêm/Xóa Foreign Key**
```javascript
// up(): Thêm foreign key
await queryInterface.addConstraint('users', {
  fields: ['role_id'],
  type: 'foreign key',
  name: 'fk_users_role',
  references: {
    table: 'roles',
    field: 'id'
  },
  onDelete: 'CASCADE',
  onUpdate: 'CASCADE'
});

// down(): Xóa foreign key
await queryInterface.removeConstraint('users', 'fk_users_role');
```

### **G. Insert/Delete dữ liệu**
```javascript
// up(): Insert dữ liệu
await queryInterface.bulkInsert('users', [
  {
    username: 'admin',
    email: 'admin@example.com',
    createdAt: new Date(),
    updatedAt: new Date()
  },
  {
    username: 'user1',
    email: 'user1@example.com',
    createdAt: new Date(),
    updatedAt: new Date()
  }
]);

// down(): Xóa dữ liệu
await queryInterface.bulkDelete('users', {
  username: ['admin', 'user1']
});
```

---

## **7. CẤU HÌNH config/config.json**

```json
{
  "development": {
    "username": "root",
    "password": "123456",
    "database": "qr_attendance_system",
    "host": "localhost",
    "port": 3306,
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": "123456",
    "database": "qr_attendance_test",
    "host": "localhost",
    "dialect": "mysql"
  },
  "production": {
    "username": "prod_user",
    "password": "strong_password",
    "database": "qr_attendance_prod",
    "host": "db.example.com",
    "dialect": "mysql",
    "logging": false
  }
}
```

### **Hoặc dùng .js với environment variables:**

```javascript
// config/config.js
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    dialect: 'mysql'
  },
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'postgres',
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false
      }
    }
  }
};
```

---

## **8. QUY TẮC VÀNG**

### ✅ **Nên làm:**
1. **Migration phải có cả up() và down()** - down() phải ngược lại với up()
2. **Không sửa migration đã chạy** - Tạo migration mới để sửa
3. **Test migration trước khi commit** - Chạy up() và down() để kiểm tra
4. **Đặt tên migration rõ ràng** - `create-users-table`, `add-email-to-users`
5. **Commit migration vào Git** - Để đồng bộ với team
6. **Chạy migration trên production cẩn thận** - Backup trước khi chạy

### ❌ **Không nên làm:**
1. ❌ Sửa file migration đã chạy (đã có trong SequelizeMeta)
2. ❌ Xóa file migration đã chạy
3. ❌ Chạy SQL trực tiếp thay vì dùng migration
4. ❌ Để down() trống hoặc không phù hợp với up()
5. ❌ Hard-code dữ liệu quan trọng trong migration
6. ❌ Skip migration bằng cách xóa SequelizeMeta

---

## **9. WORKFLOW THỰC TẾ**

### **A. Bắt đầu dự án mới**
```bash
# 1. Cài đặt
npm install sequelize sequelize-cli mysql2

# 2. Khởi tạo
npx sequelize-cli init

# 3. Sửa config/config.json

# 4. Tạo database
mysql -u root -p
CREATE DATABASE qr_attendance_system;

# 5. Tạo migration
npx sequelize-cli migration:generate --name create-tables

# 6. Viết code migration

# 7. Chạy migration
npx sequelize-cli db:migrate

# 8. Commit
git add migrations/ config/
git commit -m "Add database migrations"
git push
```

### **B. Clone project về**
```bash
# 1. Clone
git clone <repo>

# 2. Install
npm install

# 3. Tạo database
mysql -u root -p
CREATE DATABASE qr_attendance_system;

# 4. Chạy migrations
npx sequelize-cli db:migrate

# ✅ Database đã giống với team
```

### **C. Thêm feature mới cần thay đổi database**
```bash
# 1. Tạo migration
npx sequelize-cli migration:generate --name add-avatar-to-users

# 2. Viết code
# migrations/XXXXXX-add-avatar-to-users.js

# 3. Test local
npx sequelize-cli db:migrate
# Kiểm tra database

# 4. Test rollback
npx sequelize-cli db:migrate:undo
npx sequelize-cli db:migrate

# 5. Commit
git add migrations/
git commit -m "Add avatar column to users"
git push

# 6. Team pull và chạy
git pull
npx sequelize-cli db:migrate
```

---

## **10. TROUBLESHOOTING**

### **Lỗi: Table doesn't exist**
```bash
# Nguyên nhân: Chưa chạy migration tạo bảng
# Giải pháp:
npx sequelize-cli db:migrate:status  # Xem migrations nào chưa chạy
npx sequelize-cli db:migrate         # Chạy migrations
```

### **Lỗi: Migration already executed**
```bash
# Nguyên nhân: Migration đã chạy rồi
# Giải pháp: Tạo migration mới, không sửa migration cũ
```

### **Muốn chạy lại tất cả migrations**
```bash
# Rollback tất cả
npx sequelize-cli db:migrate:undo:all

# Chạy lại
npx sequelize-cli db:migrate
```

### **Sửa migration đã chạy sai**
```bash
# KHÔNG SỬA trực tiếp file migration cũ
# Thay vào đó:

# 1. Rollback
npx sequelize-cli db:migrate:undo

# 2. Sửa file migration

# 3. Chạy lại
npx sequelize-cli db:migrate

# Hoặc tạo migration mới để fix
npx sequelize-cli migration:generate --name fix-users-table
```

---

## **11. SO SÁNH MIGRATION VS SEEDER**

| | Migration | Seeder |
|---|---|---|
| **Mục đích** | Thay đổi cấu trúc DB | Thêm dữ liệu mẫu |
| **Khi nào dùng** | Tạo bảng, thêm cột, index | Demo data, test data |
| **Production** | ✅ Chạy | ❌ Không chạy (thường) |
| **Rollback** | Có (quan trọng) | Có (ít quan trọng) |
| **Ví dụ** | CREATE TABLE users | INSERT demo users |

---

## **12. BEST PRACTICES**

```javascript
// ✅ GOOD: Có down() phù hợp
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('users', { ... });
  },
  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('users');
  }
};

// ✅ GOOD: Tên migration rõ ràng
// 20250423-create-users-table.js
// 20250424-add-email-to-users.js
// 20250425-add-index-on-users-email.js

// ✅ GOOD: Transaction để đảm bảo atomic
module.exports = {
  async up(queryInterface, Sequelize) {
    const transaction = await queryInterface.sequelize.transaction();
    try {
      await queryInterface.createTable('users', { ... }, { transaction });
      await queryInterface.createTable('roles', { ... }, { transaction });
      await transaction.commit();
    } catch (err) {
      await transaction.rollback();
      throw err;
    }
  }
};

// ❌ BAD: down() trống
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('users', { ... });
  },
  async down(queryInterface, Sequelize) {
    // Trống - không rollback được!
  }
};
```

---

## **13. CHEAT SHEET - CÁC LỆNH THƯỜNG DÙNG**

```bash
# === KHỞI TẠO ===
npx sequelize-cli init                    # Tạo folders

# === MIGRATION ===
npx sequelize-cli migration:generate --name XXX   # Tạo migration
npx sequelize-cli db:migrate                      # Chạy migrations
npx sequelize-cli db:migrate:status               # Xem trạng thái
npx sequelize-cli db:migrate:undo                 # Rollback 1
npx sequelize-cli db:migrate:undo:all             # Rollback tất cả

# === SEEDER ===
npx sequelize-cli seed:generate --name XXX   # Tạo seeder
npx sequelize-cli db:seed:all                # Chạy seeders
npx sequelize-cli db:seed:undo:all           # Xóa seed data

# === DEBUG ===
npx sequelize-cli db:migrate --debug         # Chạy với debug mode
```

---

## **14. VÍ DỤ MIGRATION ĐẦY ĐỦ**

### **Ví dụ 1: Tạo bảng users và roles**

```javascript
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {
    // Tạo bảng roles trước
    await queryInterface.createTable('roles', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
        allowNull: false
      },
      name: {
        type: Sequelize.STRING(50),
        allowNull: false,
        unique: true
      },
      description: {
        type: Sequelize.STRING(255)
      },
      createdAt: {
        type: Sequelize.DATE,
        allowNull: false,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP')
      },
      updatedAt: {
        type: Sequelize.DATE,
        allowNull: false,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP')
      }
    });

    // Insert roles mặc định
    await queryInterface.bulkInsert('roles', [
      {
        id: 1,
        name: 'admin',
        description: 'Quản trị viên',
        createdAt: new Date(),
        updatedAt: new Date()
      },
      {
        id: 2,
        name: 'lecturer',
        description: 'Giảng viên',
        createdAt: new Date(),
        updatedAt: new Date()
      },
      {
        id: 3,
        name: 'student',
        description: 'Sinh viên',
        createdAt: new Date(),
        updatedAt: new Date()
      }
    ]);

    // Tạo bảng users
    await queryInterface.createTable('users', {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
        allowNull: false
      },
      username: {
        type: Sequelize.STRING(50),
        allowNull: false,
        unique: true
      },
      password: {
        type: Sequelize.STRING(255),
        allowNull: false
      },
      first_name: {
        type: Sequelize.STRING(50),
        allowNull: false
      },
      last_name: {
        type: Sequelize.STRING(50),
        allowNull: false
      },
      email: {
        type: Sequelize.STRING(100),
        unique: true
      },
      phone: {
        type: Sequelize.STRING(20)
      },
      avatar: {
        type: Sequelize.STRING(255)
      },
      role_id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {
          model: 'roles',
          key: 'id'
        },
        onUpdate: 'CASCADE',
        onDelete: 'RESTRICT'
      },
      is_active: {
        type: Sequelize.BOOLEAN,
        defaultValue: true
      },
      createdAt: {
        type: Sequelize.DATE,
        allowNull: false,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP')
      },
      updatedAt: {
        type: Sequelize.DATE,
        allowNull: false,
        defaultValue: Sequelize.literal('CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP')
      }
    });

    // Tạo indexes
    await queryInterface.addIndex('users', ['username']);
    await queryInterface.addIndex('users', ['email']);
    await queryInterface.addIndex('users', ['role_id']);
  },

  async down(queryInterface, Sequelize) {
    await queryInterface.dropTable('users');
    await queryInterface.dropTable('roles');
  }
};
```

### **Ví dụ 2: Seed admin user**

```javascript
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {
    const bcrypt = require('bcryptjs');
    
    const username = process.env.DEFAULT_ADMIN_USERNAME || 'admin';
    const password = process.env.DEFAULT_ADMIN_PASSWORD || 'admin@123';
    const hashedPassword = await bcrypt.hash(password, 10);

    await queryInterface.bulkInsert('users', [
      { 
        username: username, 
        password: hashedPassword, 
        last_name: 'Quản trị', 
        first_name: 'viên', 
        email: 'admin@example.com',
        role_id: 1,
        is_active: true,
        createdAt: new Date(),
        updatedAt: new Date()
      }
    ]);
  },

  async down(queryInterface, Sequelize) {
    const username = process.env.DEFAULT_ADMIN_USERNAME || 'admin';
    await queryInterface.bulkDelete('users', { username: username }, {});
  }
};
```

### **Ví dụ 3: Thêm cột avatar**

```javascript
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.addColumn('users', 'avatar', {
      type: Sequelize.STRING(255),
      allowNull: true,
      after: 'phone' // MySQL specific
    });
  },

  async down(queryInterface, Sequelize) {
    await queryInterface.removeColumn('users', 'avatar');
  }
};
```

---

## **15. TIMELINE MIGRATION**

```
┌─────────────────────────────────────────────────────────┐
│  npx sequelize-cli db:migrate                           │
└─────────────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│  1. Đọc config/config.json                              │
│  2. Kết nối database                                    │
│  3. Kiểm tra bảng SequelizeMeta                         │
│  4. Quét thư mục migrations/                            │
│  5. So sánh với SequelizeMeta                           │
│  6. Chạy migrations chưa thực thi (theo thứ tự)        │
│  7. Với mỗi migration:                                  │
│     - Chạy hàm up()                                     │
│     - Ghi vào SequelizeMeta                             │
│  8. Kết thúc                                            │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  npx sequelize-cli db:migrate:undo                      │
└─────────────────────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│  1. Lấy migration cuối cùng từ SequelizeMeta            │
│  2. Chạy hàm down() của migration đó                    │
│  3. Xóa khỏi SequelizeMeta                              │
│  4. Kết thúc                                            │
└─────────────────────────────────────────────────────────┘
```

---

## **16. TÓM TẮT 1 DÒNG**

```
┌──────────────────────────────────────────────────────────┐
│  init → tạo folders                                      │
│  migration:generate → tạo file (2 hàm up/down)          │
│  db:migrate → chạy up() (thay đổi DB)                   │
│  db:migrate:undo → chạy down() (rollback)               │
└──────────────────────────────────────────────────────────┘
```

---

## **17. SEQUELIZE CLI VS MONGOOSE**

| Tiêu chí | Sequelize | Mongoose |
|----------|-----------|----------|
| **Database** | SQL (MySQL, PostgreSQL, SQLite) | NoSQL (MongoDB) |
| **Schema** | **Cứng nhắc** (phải có bảng trước) | **Linh hoạt** (schema-less) |
| **Migrations** | **CÓ** (Sequelize CLI) | **KHÔNG** (không cần) |
| **Thay đổi cấu trúc** | Phải tạo migration | Chỉ cần sửa Schema |
| **Relations** | Foreign Keys thực sự | Reference ObjectId |
| **Transactions** | Có (quan trọng cho SQL) | Có (từ v4.0+) |

### **Khi nào dùng Sequelize?**
- ✅ Dữ liệu có cấu trúc rõ ràng, ít thay đổi
- ✅ Cần ACID transactions (banking, e-commerce)
- ✅ Quan hệ phức tạp giữa các bảng (JOIN nhiều)
- ✅ Cần data integrity cao (foreign keys, constraints)
- ✅ Dự án lớn, nhiều người, cần kiểm soát schema chặt chẽ

### **Khi nào dùng Mongoose?**
- ✅ Dữ liệu linh hoạt, thay đổi nhiều (startup, MVP)
- ✅ Schema không cố định (mỗi document khác nhau)
- ✅ Muốn develop nhanh (không cần migration)
- ✅ Dữ liệu dạng JSON, nested objects
- ✅ Ít quan hệ phức tạp giữa các collections

---

## **18. TÀI LIỆU THAM KHẢO**

- **Sequelize Docs:** https://sequelize.org/docs/v6/
- **Sequelize CLI Docs:** https://github.com/sequelize/cli
- **Migration Guide:** https://sequelize.org/docs/v6/other-topics/migrations/
- **Query Interface:** https://sequelize.org/api/v6/class/src/dialects/abstract/query-interface.js~queryinterface

---

🎯 **Chúc bạn học tốt Sequelize CLI!**
