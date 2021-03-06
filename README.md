<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo_text.svg" width="320" alt="Nest Logo" /></a>
</p>

[travis-image]: https://api.travis-ci.org/nestjs/nest.svg?branch=master
[travis-url]: https://travis-ci.org/nestjs/nest
[linux-image]: https://img.shields.io/travis/nestjs/nest/master.svg?label=linux
[linux-url]: https://travis-ci.org/nestjs/nest
  
  <p align="center">A progressive <a href="http://nodejs.org" target="blank">Node.js</a> framework for building efficient and scalable server-side applications, heavily inspired by <a href="https://angular.io" target="blank">Angular</a>.</p>
    <p align="center">
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/v/@nestjs/core.svg" alt="NPM Version" /></a>
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/l/@nestjs/core.svg" alt="Package License" /></a>
<a href="https://www.npmjs.com/~nestjscore"><img src="https://img.shields.io/npm/dm/@nestjs/core.svg" alt="NPM Downloads" /></a>
<a href="https://travis-ci.org/nestjs/nest"><img src="https://api.travis-ci.org/nestjs/nest.svg?branch=master" alt="Travis" /></a>
<a href="https://travis-ci.org/nestjs/nest"><img src="https://img.shields.io/travis/nestjs/nest/master.svg?label=linux" alt="Linux" /></a>
<a href="https://coveralls.io/github/nestjs/nest?branch=master"><img src="https://coveralls.io/repos/github/nestjs/nest/badge.svg?branch=master#5" alt="Coverage" /></a>
<a href="https://gitter.im/nestjs/nestjs?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=body_badge"><img src="https://badges.gitter.im/nestjs/nestjs.svg" alt="Gitter" /></a>
<a href="https://opencollective.com/nest#backer"><img src="https://opencollective.com/nest/backers/badge.svg" alt="Backers on Open Collective" /></a>
<a href="https://opencollective.com/nest#sponsor"><img src="https://opencollective.com/nest/sponsors/badge.svg" alt="Sponsors on Open Collective" /></a>
  <a href="https://paypal.me/kamilmysliwiec"><img src="https://img.shields.io/badge/Donate-PayPal-dc3d53.svg"/></a>
  <a href="https://twitter.com/nestframework"><img src="https://img.shields.io/twitter/follow/nestframework.svg?style=social&label=Follow"></a>
</p>
  <!--[![Backers on Open Collective](https://opencollective.com/nest/backers/badge.svg)](https://opencollective.com/nest#backer)
  [![Sponsors on Open Collective](https://opencollective.com/nest/sponsors/badge.svg)](https://opencollective.com/nest#sponsor)-->

## Description

[Nest](https://github.com/nestjs/nest) framework TypeScript starter repository.

## Installation

```bash
$ git clone https://github.com/Bouabidi/nestgraphql.git
```
```bash
$ cd nestgraphql
```

```bash
$ npm install
```

## Running the app

```bash
# start Mongodb
```

```bash
# open a terminal and run the server
$ npm run start:dev
```

## Test

```bash
# unit tests
$ npm run test

# e2e tests
$ npm run test:e2e

# test coverage
$ npm run test:cov
```

## Tutorial

1. nest new nestgraphql

2. cd nestgraphql

3. npm i --save @nestjs/graphql apollo-server-express graphql-tools graphql @nestjs/mongoose mongoose type-graphql


4. nest g module items
5. nest g service items
6. nest g resolver items


in items folder add the following:


1. add createitem.dto.ts
2. add item.interface.ts
3. add inputitems.input.ts
4. add item.schema.ts
5. add item.graphql


### update AppModule and import GraphQLModule

```javascript

import { GraphQLModule } from '@nestjs/graphql';
@Module({
 imports: [
  GraphQLModule.forRoot({
   autoSchemaFile: 'schema.gql',
  })]
export class AppModule {}
```

### add the connection to the database by importing MongooseModule

```javascript
import { MongooseModule } from '@nestjs/mongoose';

@Module({
 imports: [
  MongooseModule.forRoot('mongodb://localhost/nestgraphql')
 ],
})
export class AppModule {}
```

### update item.schema.ts

```javascript
import * as mongoose from 'mongoose';

export const ItemSchema = new mongoose.Schema({
 title: String,
 price: Number,
 description: String,
});
```

### update item.interface.ts


```javascript
import { Document } from 'mongoose';

export interface Item extends Document {
 readonly title: string;
 readonly price: number;
 readonly description: string;
}
```

### update createitem.dto.ts 

```javascript
import { ObjectType, Field, Int, ID } from 'type-graphql';

@ObjectType()
export class ItemType {
  @Field(() => ID)
  readonly id?: string;
  @Field()
  readonly title: string;
  @Field(() => Int)
  readonly price: number;
  @Field()
  readonly description: string;
}
```

### update inputitems.input.ts

```javascript

import { InputType, Field, Int } from 'type-graphql';

@InputType()
export class ItemInput {
  @Field()
  readonly title: string;
  @Field(() => Int)
  readonly price: number;
  @Field()
  readonly description: string;
}
```

### create database:


import our schema into ItemsModule
```javascript
@Module({
 imports: [MongooseModule.forFeature([{ name: 'Item', schema: ItemSchema }])],
providers: [ItemsResolver, ItemsService],
})
export class ItemsModule {}
```

### implement crud with GraphQL

--first create crud functionality 
there for we will update our service
we will import the database model and implement functionality using Mongodb functions.

```javascript
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Model } from 'mongoose';
import { ItemType } from './dto/create-item.dto';
import { Item } from './interfaces/item.interface';
import { ItemInput } from './input-items.input';

@Injectable()
export class ItemsService {
  constructor(@InjectModel('Item') private itemModel: Model<Item>) {}

  async create(createItemDto: ItemInput): Promise<ItemType> {
    const createdItem = new this.itemModel(createItemDto);
    return await createdItem.save();
  }

  async findAll(): Promise<ItemType[]> {
    return await this.itemModel.find().exec();
  }

  async findOne(id: string): Promise<ItemType> {
    return await this.itemModel.findOne({ _id: id });
  }

  async delete(id: string): Promise<ItemType> {
    return await this.itemModel.findByIdAndRemove(id);
  }

  async update(id: string, item: Item): Promise<ItemType> {
    return await this.itemModel.findByIdAndUpdate(id, item, { new: true });
  }
}
```

---resolver
now we can implement our Resolver. Resolver can be seen as a controller 
in a crud functionality without GraphQL.
Querys and Mutations will be defined.

```javascript
import { Resolver, Query, Mutation, Args } from '@nestjs/graphql';
import { ItemsService } from './items.service';
import { ItemType } from './dto/create-item.dto';
import { ItemInput } from './input-items.input';

@Resolver()
export class ItemsResolver {
  constructor(private readonly itemsService: ItemsService) {}

  @Query(() => [ItemType])
  async items(): Promise<ItemType[]> {
    return this.itemsService.findAll();
  }

  @Mutation(() => ItemType)
  async createItem(@Args('input') input: ItemInput): Promise<ItemInput> {
    return this.itemsService.create(input);
  }

  @Mutation(() => ItemType)
  async updateItem(
    @Args('id') id: string,
    @Args('input') input: ItemInput,
  ): Promise<ItemInput> {
    return this.itemsService.update(id, input);
  }

  @Mutation(() => ItemType)
  async deleteItem(@Args('id') id: string): Promise<ItemInput> {
    return this.itemsService.delete(id);
  }

  @Query(() => String)
  async hello() {
    return 'hello';
  }
}
```
---item.graphql
update the file 
```javascript
input ItemInput {
  title: String!
  price: Int!
  description: String!
}

type ItemType {
  id: ID!
  title: String!
  price: Int!
  description: String!
}

type Mutation {
  createItem(input: ItemInput!): ItemType!
  updateItem(input: ItemInput!, id: String!): ItemType!
  deleteItem(id: String!): ItemType!
}

type Query {
  items: [ItemType!]!
  hello: String!
}

```



