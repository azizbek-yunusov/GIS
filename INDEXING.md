# TypeORM Index va Migration - To'liq Qo'llanma 📚

Ajoyib savol! Keling **Index** va **Migration** ni chuqur o'rganamiz.

---

## 1️⃣ INDEX nima va nima uchun kerak? 🔍

### Oddiy tushuntirish - Kitobxona misoli:

Tasavvur qiling, 10,000 ta kitob bor. Siz "Harry Potter" kitobini qidiryapsiz:

**Index yo'q:**
- Har bir kitobni birma-bir ko'rib chiqasiz
- 10,000 ta kitobni tekshirish kerak
- ⏱️ 2 soat vaqt ketadi

**Index bor:**
- Alifbo tartibidagi katalog bor
- "H" harfini topasiz → "Harry Potter" ni topasiz
- ⚡ 2 daqiqa ichida topiladi!

### Database'da:

```typescript
// INDEX YO'Q - 1,000,000 ta qatorni scan qiladi 😱
SELECT * FROM departments WHERE keyword = 'hr';
// Execution time: 5000ms

// INDEX BOR - Direct access 🚀
SELECT * FROM departments WHERE keyword = 'hr';
// Execution time: 5ms
```

---

## 2️⃣ TypeORM'da Index turlari va ishlatilishi

### A) **Simple Index** - Bitta field uchun

```typescript
@Entity('departments')
export class Department {
  @Column()
  @Index() // ← Shunchaki @Index() yozish
  keyword: string;
}
```

**Database'da nima bo'ladi:**
```sql
CREATE INDEX idx_department_keyword ON departments(keyword);
```

**Qachon ishlatiladi:**
```typescript
// Bu query'lar tez ishlaydi:
await departmentRepo.findOne({ where: { keyword: 'hr' } });
await departmentRepo.find({ where: { keyword: Like('%engineering%') } });
```

---

### B) **Unique Index** - Takrorlanmaydigan qiymatlar

```typescript
@Entity('departments')
export class Department {
  @Column()
  @Index({ unique: true }) // ← Unique constraint
  keyword: string;
}
```

**Database'da:**
```sql
CREATE UNIQUE INDEX idx_department_keyword ON departments(keyword);
```

**Foyda:**
```typescript
// Database level'da tekshiradi - 2 ta bir xil keyword bo'lolmaydi
const dept1 = await departmentRepo.save({ keyword: 'hr', name: 'HR' });
const dept2 = await departmentRepo.save({ keyword: 'hr', name: 'HR2' }); 
// ❌ Error: duplicate key value violates unique constraint
```

---

### C) **Composite Index** - Ko'p field uchun

```typescript
@Entity('departments')
@Index(['keyword', 'isActive']) // ← Bir necha field birgalikda
export class Department {
  @Column()
  keyword: string;

  @Column()
  isActive: boolean;
}
```

**Database'da:**
```sql
CREATE INDEX idx_department_keyword_isActive 
ON departments(keyword, isActive);
```

**Qachon tez ishlaydi:**
```typescript
// ✅ TEZ - index ishlatadi
await departmentRepo.find({ 
  where: { keyword: 'hr', isActive: true } 
});

// ✅ TEZ - index'ning birinchi qismi ishlatiladi
await departmentRepo.find({ 
  where: { keyword: 'hr' } 
});

// ⚠️ SEKIN - index ishlamaydi (keyword yo'q)
await departmentRepo.find({ 
  where: { isActive: true } 
});
```

**Index tartibi muhim!** 📌

---

### D) **Partial Index** - Shartli index (Eng kuchli!)

```typescript
@Entity('departments')
@Index('idx_active_departments', ['keyword'], {
  unique: true,
  where: '"is_deleted" = false' // ← Faqat o'chirilmaganlar uchun
})
export class Department {
  @Column()
  keyword: string;

  @Column({ name: 'is_deleted' })
  isDeleted: boolean;
}
```

**Database'da:**
```sql
CREATE UNIQUE INDEX idx_active_departments 
ON departments(keyword) 
WHERE is_deleted = false;
```

**Foyda:**
```typescript
// ✅ ISHLAYDI - keyword unique faqat active'lar uchun
await departmentRepo.save({ keyword: 'hr', isDeleted: false });
await departmentRepo.save({ keyword: 'hr', isDeleted: false }); 
// ❌ Error

// ✅ ISHLAYDI - deleted'lar uchun unique emas
await departmentRepo.softRemove({ keyword: 'hr' }); // isDeleted = true
await departmentRepo.save({ keyword: 'hr', isDeleted: false }); // ✅ OK!

// Bir necha marta o'chirish va yaratish mumkin
await departmentRepo.save({ keyword: 'hr', isDeleted: true }); // ✅ OK
await departmentRepo.save({ keyword: 'hr', isDeleted: true }); // ✅ OK
await departmentRepo.save({ keyword: 'hr', isDeleted: true }); // ✅ OK
```

**Bu SOFT DELETE uchun ideal yechim!** 🎯

---

### E) **Named Index** - O'zingiz nom bering

```typescript
@Entity('departments')
@Index('idx_dept_keyword_unique', ['keyword'], { unique: true })
@Index('idx_dept_active_search', ['isActive', 'isDeleted'])
export class Department {
  @Column()
  keyword: string;

  @Column()
  isActive: boolean;

  @Column()
  isDeleted: boolean;
}
```

**Nima uchun nom berish kerak:**
- Migration'da boshqarish oson
- Index'ni o'chirish/o'zgartirish kerak bo'lganda topish oson
- Xatoliklarni tushunish oson

---

## 3️⃣ Index Performance - Real misollar

### Benchmark test (1,000,000 ta qator):

```typescript
// TEST 1: Index yo'q
@Entity()
export class Department {
  @Column()
  keyword: string; // Index yo'q
}

// Query:
await departmentRepo.findOne({ where: { keyword: 'engineering' } });
// ⏱️ Result: 4,500ms (FULL TABLE SCAN)
// Database: Scanned 1,000,000 rows

// TEST 2: Index bor
@Entity()
export class Department {
  @Column()
  @Index()
  keyword: string; // Index mavjud
}

// Query:
await departmentRepo.findOne({ where: { keyword: 'engineering' } });
// ⚡ Result: 3ms (INDEX SEEK)
// Database: Scanned 1 row

// 1,500x TEZROQ! 🚀
```

---

## 4️⃣ MIGRATION - Nima va nima uchun? 🔄

### Migration nima?

**Migration** - Bu database schema'sini o'zgartirish uchun versiyalangan SQL script.

### Misol: Loyiha rivojlanishi

```
Day 1: Database yaratish
  ↓
Day 5: Yangi column qo'shish
  ↓
Day 10: Index qo'shish
  ↓
Day 20: Foreign key qo'shish
  ↓
Day 30: Column o'chirish
```

Har bir o'zgarish = 1 ta migration file

---

### Migration'siz muammo ❌

```typescript
// Developer 1 (local database)
@Entity()
export class Department {
  @Column()
  name: string;
}

// Developer 2 (local database) - 1 hafta o'tgach
@Entity()
export class Department {
  @Column()
  name: string;
  
  @Column() // Yangi field qo'shdi
  description: string;
}

// Production server -eski versiya
// ❌ Code deploy qilganda ERROR:
// Column 'description' not found in database
```

### Migration bilan ✅

```typescript
// Migration 1: Initial
export class CreateDepartments1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      CREATE TABLE departments (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL
      )
    `);
  }
}

// Migration 2: Add description
export class AddDescriptionToDepartments1234567891 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE departments 
      ADD COLUMN description TEXT
    `);
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE departments 
      DROP COLUMN description
    `);
  }
}

// Har bir developer va production:
// npm run migration:run
// ✅ Hamma database bir xil holatda!
```

---

## 5️⃣ Index uchun Migration yaratish - Step by step

### Step 1: TypeORM Config

```typescript
// data-source.ts
import { DataSource } from 'typeorm';

export const AppDataSource = new DataSource({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'postgres',
  password: 'password',
  database: 'hr_recruitment',
  entities: ['src/**/*.entity.ts'],
  migrations: ['src/migrations/*.ts'],
  synchronize: false, // ❌ Production'da false!
});
```

**⚠️ MUHIM:** `synchronize: true` - faqat development'da!

---

### Step 2: Entity yaratish

```typescript
// department.entity.ts
@Entity('departments')
@Index('idx_dept_keyword_unique', ['keyword'], { 
  unique: true,
  where: '"is_deleted" = false'
})
@Index('idx_dept_search', ['isActive', 'isDeleted'])
export class Department {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 50 })
  keyword: string;

  @Column({ name: 'is_active', default: true })
  isActive: boolean;

  @Column({ name: 'is_deleted', default: false })
  isDeleted: boolean;
}
```

---

### Step 3: Migration generate qilish

```bash
# TypeORM avtomatik migration yaratadi
npm run typeorm migration:generate -- -n AddIndexesToDepartments

# Yoki qo'lda yaratish:
npm run typeorm migration:create -- -n AddIndexesToDepartments
```

---

### Step 4: Migration file (avtomatik yaratilgan)

```typescript
// migrations/1703345678901-AddIndexesToDepartments.ts
import { MigrationInterface, QueryRunner } from 'typeorm';

export class AddIndexesToDepartments1703345678901 implements MigrationInterface {
  name = 'AddIndexesToDepartments1703345678901';

  public async up(queryRunner: QueryRunner): Promise<void> {
    // Index yaratish
    await queryRunner.query(`
      CREATE UNIQUE INDEX idx_dept_keyword_unique 
      ON departments(keyword) 
      WHERE is_deleted = false
    `);
    
    await queryRunner.query(`
      CREATE INDEX idx_dept_search 
      ON departments(is_active, is_deleted)
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    // Rollback - index o'chirish
    await queryRunner.query(`
      DROP INDEX IF EXISTS idx_dept_keyword_unique
    `);
    
    await queryRunner.query(`
      DROP INDEX IF EXISTS idx_dept_search
    `);
  }
}
```

---

### Step 5: Migration run qilish

```bash
# Migration'larni bajarish
npm run migration:run

# Output:
# ✅ Migration AddIndexesToDepartments1703345678901 has been executed

# Agar xato bo'lsa - rollback
npm run migration:revert

# Output:
# ✅ Migration AddIndexesToDepartments1703345678901 has been reverted
```

---

## 6️⃣ Real Loyiha - To'liq Misol

### package.json

```json
{
  "scripts": {
    "typeorm": "typeorm-ts-node-commonjs",
    "migration:generate": "npm run typeorm -- migration:generate",
    "migration:create": "npm run typeorm -- migration:create",
    "migration:run": "npm run typeorm -- migration:run -d src/data-source.ts",
    "migration:revert": "npm run typeorm -- migration:revert -d src/data-source.ts",
    "migration:show": "npm run typeorm -- migration:show -d src/data-source.ts"
  }
}
```

---

### Migration History Jadvali

TypeORM avtomatik `migrations` jadval yaratadi:

```sql
SELECT * FROM migrations;

| id | timestamp       | name                                    |
|----|-----------------|----------------------------------------|
| 1  | 1703345678901  | CreateDepartments1703345678901         |
| 2  | 1703345678902  | AddIndexesToDepartments1703345678902   |
| 3  | 1703345678903  | AddDeletedByColumn1703345678903        |
```

Har bir migration faqat **bir marta** ishga tushadi!

---

## 7️⃣ Qo'shimcha Index strategiyalari

### A) Covering Index - Super tez!

```typescript
@Entity('departments')
@Index('idx_dept_covering', ['keyword', 'name', 'isActive'])
export class Department {
  @Column()
  keyword: string;

  @Column()
  name: string;

  @Column()
  isActive: boolean;
}

// Bu query faqat index'dan o'qiydi - table'ga kirmaydi!
await departmentRepo.find({
  where: { keyword: 'hr' },
  select: ['keyword', 'name', 'isActive']
});
// ⚡ Super tez - 1ms
```

---

### B) Full-Text Search Index

```typescript
@Entity('departments')
export class Department {
  @Column()
  name: string;

  @Column()
  description: string;
}

// Migration'da:
export class AddFullTextSearch1234567890 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    // PostgreSQL full-text search
    await queryRunner.query(`
      ALTER TABLE departments 
      ADD COLUMN search_vector tsvector 
      GENERATED ALWAYS AS (
        to_tsvector('english', name || ' ' || description)
      ) STORED
    `);
    
    await queryRunner.query(`
      CREATE INDEX idx_departments_search 
      ON departments USING GIN (search_vector)
    `);
  }
}

// Ishlatish:
const results = await queryRunner.query(`
  SELECT * FROM departments 
  WHERE search_vector @@ to_tsquery('english', 'software & engineer')
`);
```

---

## 8️⃣ Index Monitoring - Performance tekshirish

### Index ishlatyaptimi? 🤔

```sql
-- PostgreSQL
EXPLAIN ANALYZE 
SELECT * FROM departments 
WHERE keyword = 'hr';

-- Output:
-- Index Scan using idx_dept_keyword on departments
-- Cost: 0.15..8.17 rows=1
-- Execution time: 0.045ms ✅

-- vs

-- Seq Scan on departments
-- Cost: 0.00..18.50 rows=1
-- Execution time: 2.500ms ❌
```

---

### TypeORM'da monitoring:

```typescript
// Query logging yoqish
@Module({
  imports: [
    TypeOrmModule.forRoot({
      // ...
      logging: ['query', 'error', 'schema'],
      logger: 'advanced-console',
    }),
  ],
})

// Yoki specific query uchun:
const query = departmentRepo
  .createQueryBuilder('dept')
  .where('dept.keyword = :keyword', { keyword: 'hr' });

console.log(query.getSql()); // SQL ko'rish
const results = await query.getMany();
```

---

## 9️⃣ Index Best Practices ⭐

### ✅ DO - Qilish kerak:

```typescript
// 1. WHERE clause'da ko'p ishlatiladigan fieldlar
@Index()
@Column()
keyword: string;

// 2. Foreign key'lar (relation'lar)
@Index()
@ManyToOne(() => User)
createdBy: User;

// 3. Sort/Order by qilinadigan fieldlar
@Index()
@Column()
createdAt: Date;

// 4. Unique constraint'lar
@Index({ unique: true })
@Column()
email: string;
```

### ❌ DON'T - Qilmaslik kerak:

```typescript
// 1. Kam ishlatiladigan fieldlar
@Index() // ❌ Ortiqcha
@Column()
internalNotes: string;

// 2. Boolean fieldlar (faqat 2 ta qiymat)
@Index() // ❌ Unchalik foydasi yo'q
@Column()
isActive: boolean;

// 3. Juda ko'p index
// Har bir index - disk joy + insert/update sekinlashadi
```

---

## 🎯 Xulosa

### Index:
- **Nima:** Database'da tez qidiruv uchun "katalog"
- **Nima uchun:** Query performance 100-1000x yaxshilaydi
- **Qachon:** WHERE, JOIN, ORDER BY'da ishlatiladigan fieldlar
- **Cost:** Disk joy + insert/update biroz sekinlashadi

### Migration:
- **Nima:** Database schema o'zgarishlarini boshqarish
- **Nima uchun:** Version control, team work, production safety
- **Qachon:** Entity o'zgarganda, index qo'shganda, har qanday schema o'zgarishida
- **Cost:** Vaqt sarflanadi, lekin xatolarni kamaytiradi

**Golden Rule:** Index'lar - bu kitobxonadagi katalog. Kerakli joyda - juda foydali, ortiqcha bo'lsa - joy band qiladi! 📚

Savol bormi? 😊
