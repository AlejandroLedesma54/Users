# Users

1. Configuración del proyecto
Estructura del Proyecto
lua
Copy code
ecommerce-app/
│
├── src/
│   ├── controllers/
│   ├── models/
│   ├── routes/
│   ├── middleware/
│   ├── utils/
│   ├── config/
│   └── app.ts
├── .env
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
2. Inicialización del proyecto
bash
Copy code
# Crear la carpeta del proyecto
mkdir ecommerce-app && cd ecommerce-app

# Inicializar npm
npm init -y

# Instalar dependencias
npm install express sequelize sequelize-typescript mysql2 dotenv jsonwebtoken bcryptjs

# Instalar dependencias de desarrollo
npm install typescript ts-node @types/express @types/jsonwebtoken @types/bcryptjs @types/node nodemon --save-dev

# Inicializar TypeScript
npx tsc --init
3. Configuración de TypeScript (tsconfig.json)
json
Copy code
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules"]
}
4. Configuración de Sequelize y la base de datos
Configuración del archivo .env
env

DB_HOST=localhost
DB_USER=root
DB_PASSWORD=password
DB_NAME=ecommerce_db
JWT_SECRET=your_jwt_secret
Configuración de la base de datos (src/config/database.ts)
typescript
Copy code
import { Sequelize } from 'sequelize-typescript';
import dotenv from 'dotenv';
import { User, Role, Product, Cart, ProductCart, Order, Entity, Permission } from '../models';

dotenv.config();

const sequelize = new Sequelize({
  dialect: 'mysql',
  host: process.env.DB_HOST,
  username: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  models: [User, Role, Product, Cart, ProductCart, Order, Entity, Permission],
  logging: false,
});

export default sequelize;
5. Creación de modelos
Modelo User (src/models/User.ts)
typescript
Copy code
import { Table, Column, Model, ForeignKey, BelongsTo, HasMany } from 'sequelize-typescript';
import { Role, Cart, Order } from './';

@Table
export class User extends Model {
  @Column
  email!: string;

  @Column
  password!: string;

  @ForeignKey(() => Role)
  @Column
  roleId!: number;

  @BelongsTo(() => Role)
  role!: Role;

  @HasMany(() => Cart)
  carts!: Cart[];

  @HasMany(() => Order)
  orders!: Order[];
}
Modelo Role (src/models/Role.ts)
typescript
Copy code
import { Table, Column, Model, HasMany } from 'sequelize-typescript';
import { User, Permission } from './';

@Table
export class Role extends Model {
  @Column
  name!: string;

  @HasMany(() => User)
  users!: User[];

  @HasMany(() => Permission)
  permissions!: Permission[];
}
Modelo Product (src/models/Product.ts)
typescript
Copy code
import { Table, Column, Model, HasMany } from 'sequelize-typescript';
import { ProductCart } from './';

@Table
export class Product extends Model {
  @Column
  name!: string;

  @Column
  price!: number;

  @Column
  description!: string;

  @Column
  stock!: number;

  @HasMany(() => ProductCart)
  productCarts!: ProductCart[];
}
Modelo Cart (src/models/Cart.ts)
typescript
Copy code
import { Table, Column, Model, ForeignKey, BelongsTo, HasMany } from 'sequelize-typescript';
import { User, ProductCart } from './';

@Table
export class Cart extends Model {
  @ForeignKey(() => User)
  @Column
  userId!: number;

  @BelongsTo(() => User)
  user!: User;

  @HasMany(() => ProductCart)
  productCarts!: ProductCart[];
}
Modelo ProductCart (src/models/ProductCart.ts)
typescript
Copy code
import { Table, Column, Model, ForeignKey, BelongsTo } from 'sequelize-typescript';
import { Product, Cart } from './';

@Table
export class ProductCart extends Model {
  @ForeignKey(() => Cart)
  @Column
  cartId!: number;

  @BelongsTo(() => Cart)
  cart!: Cart;

  @ForeignKey(() => Product)
  @Column
  productId!: number;

  @BelongsTo(() => Product)
  product!: Product;

  @Column
  quantity!: number;
}
Modelo Order (src/models/Order.ts)
typescript
Copy code
import { Table, Column, Model, ForeignKey, BelongsTo } from 'sequelize-typescript';
import { User, ProductCart } from './';

@Table
export class Order extends Model {
  @ForeignKey(() => User)
  @Column
  userId!: number;

  @BelongsTo(() => User)
  user!: User;

  @ForeignKey(() => ProductCart)
  @Column
  productCartId!: number;

  @BelongsTo(() => ProductCart)
  productCart!: ProductCart;

  @Column
  total!: number;
}
Modelo Entity (src/models/Entity.ts)
typescript
Copy code
import { Table, Column, Model, HasMany } from 'sequelize-typescript';
import { Permission } from './';

@Table
export class Entity extends Model {
  @Column
  name!: string;

  @HasMany(() => Permission)
  permissions!: Permission[];
}
Modelo Permission (src/models/Permission.ts)
typescript
Copy code
import { Table, Column, Model, ForeignKey, BelongsTo } from 'sequelize-typescript';
import { Role, Entity } from './';

@Table
export class Permission extends Model {
  @ForeignKey(() => Role)
  @Column
  roleId!: number;

  @BelongsTo(() => Role)
  role!: Role;

  @ForeignKey(() => Entity)
  @Column
  entityId!: number;

  @BelongsTo(() => Entity)
  entity!: Entity;

  @Column
  canCreate!: boolean;

  @Column
  canUpdate!: boolean;

  @Column
  canDelete!: boolean;

  @Column
  canGet!: boolean;
}
6. Configuración de la autenticación JWT
Middleware de autenticación (src/middleware/auth.ts)
typescript
Copy code
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import dotenv from 'dotenv';

dotenv.config();

const secret = process.env.JWT_SECRET as string;

export const authenticateJWT = (req: Request, res: Response, next: NextFunction) => {
  const token = req.header('Authorization')?.split(' ')[1];

  if (token) {
    jwt.verify(token, secret, (err, user) => {
      if (err) {
        return res.sendStatus(403);
      }
      req.user = user;
      next();
    });
  } else {
    res.sendStatus(401);
  }
};
Hash de contraseñas (src/utils/hashPassword.ts)
typescript
Copy code
import bcrypt from 'bcryptjs';

export const hashPassword = async (password: string): Promise<string> => {
  const salt = await bcrypt.genSalt(10);
  return await bcrypt.hash(password, salt);
};

export const comparePassword = async (password: string, hashedPassword: string): Promise<boolean> => {
  return await bcrypt.compare(password, hashedPassword);
};
7. Creación de controladores
Controlador de usuarios (src/controllers/userController.ts)
typescript
Copy code
import { Request, Response } from 'express';
import { User } from '../models/User';
import { hashPassword, comparePassword } from '../utils/hashPassword';
import jwt from 'jsonwebtoken';
import dotenv from 'dotenv';

dotenv.config();
const secret = process.env.JWT_SECRET as string;

export const createUser = async (req: Request, res: Response) => {
  const { email, password, roleId } = req.body;
  try {
    const hashedPassword = await hashPassword(password);
    const user = await User.create({ email, password: hashedPassword, roleId });
    res.status(201).json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

export const loginUser = async (req: Request, res: Response) => {
  const { email, password } = req.body;
  try {
    const user = await User.findOne({ where: { email } });
    if (user && await comparePassword(password, user.password)) {
      const token = jwt.sign({ userId: user.id, roleId: user.roleId }, secret, { expiresIn: '1h' });
      res.status(200).json({ token });
    } else {
      res.status(401).json({ error: 'Invalid email or password' });
    }
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Otros métodos CRUD (deleteUser, updateUser, getAllUsers)...
8. Creación de rutas
Rutas de usuarios (src/routes/userRoutes.ts)
typescript
Copy code
import { Router } from 'express';
import { createUser, loginUser } from '../controllers/userController';
import { authenticateJWT } from '../middleware/auth';

const router = Router();

router.post('/register', createUser);
router.post('/login', loginUser);

// Otras rutas CRUD (deleteUser, updateUser, getAllUsers)...

export default router;
9. Configuración del servidor (src/app.ts)
typescript
Copy code
import express from 'express';
import dotenv from 'dotenv';
import sequelize from './config/database';
import userRoutes from './routes/userRoutes';

dotenv.config();

const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());
app.use('/users', userRoutes);

// Otras rutas...

sequelize.sync().then(() => {
  app.listen(port, () => {
    console.log(`Server running on port ${port}`);
  });
}).catch((error) => {
  console.error('Unable to connect to the database:', error);
});
10. Ejecutar el proyecto
Para iniciar el proyecto, puedes usar el siguiente comando en la terminal:

bash
Copy code
npx nodemon src/app.ts
