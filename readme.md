# Instruções para a aula

### Instalação do sequelize

```javascript
npm install sequelize sequelize-cli mysql2
```

### Configuração da conexão com banco de dados

- Criar arquivo com as configurações `src/config/database.js`

```javascript
module.exports = {
  dialect: "mysql",
  host: "localhost",
  username: "root",
  password: "****",
  database: "syscondom",
  logging: false,
  define: {
    timestamp: true,
    underscored: true,
  },
};
```

- Criar arquivo de configuração sequelize `/.sequelizerc` (na raíz do projeto)

> Aqui dizemos para o sequelize onde procurar nossos arquivos de configuração da conexão e das migrations.

```javascript
const path = require("path");

module.exports = {
  config: path.resolve(__dirname, "src", "config", "database.js"),
  "migrations-path": path.resolve(__dirname, "src", "database", "migrations"),
};
```

- Criar arquivo de conexão `src/database/index.js`

```javascript
const Sequelize = require("sequelize");
const dbConfig = require("../config/database");
const connection = new Sequelize(dbConfig);

module.exports = connection;
```

- Criar o banco de dados `npx sequelize db:create`

### Criação das migrations (estrutura do bd)

> **Migrations:** Uma forma de versionar a criação do banco de dados e sua evolução. Cria uma linha do tempo com as alterações.

- Criar migrations
  `npx sequelize migration:generate --name create_manager`

`src/database/migrations/[DATE_TIME]create_manager`

```javascript
"use strict";

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("managers", {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
        allowNull: false,
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      cpf: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      cellphone: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      password: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      created_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updated_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("managers");
  },
};
```

`src/database/migrations/[DATE_TIME]create_condominium`

`npx sequelize migration:generate --name create_condominium`

```javascript
"use strict";

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable("condominiums", {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
        allowNull: false,
      },
      created_manager_id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: { model: "managers", key: "id" },
        onUpdate: "CASCADE",
        onDelete: "CASCADE",
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      cnpj: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      ticket: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      created_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updated_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable("condominiums");
  },
};
```

- Executar migrations
  `npx sequelize db:migrate`

# Criação dos models

- Criar arquivo `src/models/Manager.js`

```javascript
const { Model, DataTypes } = require("sequelize");

class Manager extends Model {
  static init(sequelize) {
    super.init(
      {
        name: DataTypes.STRING,
        cpf: DataTypes.STRING,
        email: DataTypes.STRING,
        cellphone: DataTypes.STRING,
        password: DataTypes.STRING,
      },
      {
        sequelize,
      }
    );
  }

  static associate(models) {
    this.belongsToMany(models.Condominium, {
      foreignKey: "manager_id",
      through: models.Management,
      as: "condominium",
    });
  }
}

module.exports = Manager;
```

- Criar arquivo `src/models/Condominium.js`

```javascript
const { Model, DataTypes } = require("sequelize");

class Condominium extends Model {
  static init(sequelize) {
    super.init(
      {
        name: DataTypes.STRING,
        cnpj: DataTypes.STRING,
        ticket: DataTypes.STRING,
      },
      {
        sequelize,
        modelName: "condominiums",
      }
    );
  }

  static associate(models) {
    this.belongsTo(models.Manager, {
      foreignKey: "created_manager_id",
      as: "created_manager",
    });
    this.belongsToMany(models.Manager, {
      foreignKey: "condominium_id",
      through: models.Management,
      as: "managers",
    });
  }
}

module.exports = Condominium;
```

- Criar arquivo `src/models/Management.js`

```javascript
const { Model, DataTypes } = require("sequelize");

class Management extends Model {
  static init(sequelize) {
    super.init(
      {
        active: DataTypes.BOOLEAN,
        principal: DataTypes.BOOLEAN,
      },
      {
        sequelize,
        tableName: "managements",
      }
    );
  }

  static associate(models) {
    this.belongsTo(models.Manager, {
      foreignKey: "created_manager_id",
      as: "created_manager",
    });
  }
}

module.exports = Management;
```

- Modificar o arquivo `src/database/index.js`

  > Aqui estamos inicializando nossos models e as associações entre eles

```javascript
const Sequelize = require("sequelize");
const dbConfig = require("../config/database");

const Manager = require("../models/Manager");
const Condominium = require("../models/Condominium");
const Management = require("../models/Management");

const connection = new Sequelize(dbConfig);

Manager.init(connection);
Management.init(connection);
Condominium.init(connection);
Condominium.associate(connection.models);
Management.associate(connection.models);

module.exports = connection;
```
