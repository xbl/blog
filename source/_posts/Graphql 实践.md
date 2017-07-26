---
title: GraphQL 实践
date: 2016-11-28 18:44:48
tags: 
---

* 简介
* 数据库链接
* 写法一
* 写法二
* Introspection
* 总结

## 简介

​	[Graphql](http://graphql.org/) 是由Facebook开源的一种描述语言， 可以更清晰的描述API请求参数与返回值 ，很多其他介绍里会写是代替REST的另一种选择，其实Graphql不仅可以用在HTTP也可以用在Socket通讯或者是RPC，客户端查询你的API只返回自己需要的内容，服务端只要将API做的足够丰富，客户端UI变化就可以轻松应对。Facebook移动app和[Github API](https://developer.github.com/early-access/) 等都在使用。



## 数据库链接

​	为了可以清晰的展示出`Graphql`的能力，如何与数据配合才是它的关键所在，我们采用了Postgresql作为数据支持，后端语言使用`Nodejs`为了方便演示，`Graphql`与后端语言关系不大，无论是`Java`还是`Ruby`都可以使用。

**db.js**

```javascript
var Sequelize = require('Sequelize');

var sequelize = new Sequelize('game', 'dbuser', '123456', {
	host: 'localhost',
	dialect: 'postgres',
	pool: {
		max: 5,
		min: 0,
		idle: 10000
	},
});
// 定义city model
sequelize.define('city', {
	id: {
		type: Sequelize.STRING,
		primaryKey: true
	},
	provinceId: {
		type: Sequelize.STRING,
		field: 'province_id' 
	},
	cityId: {
		type: Sequelize.STRING,
		field: 'city_id' 
	},
	name: {
		type: Sequelize.STRING
	}
}, {
	freezeTableName: true,
	createdAt: false,
	updatedAt: false
});
// 定义store model
sequelize.define('store', {
	id: {
		type: Sequelize.STRING,
		primaryKey: true
	},
	storeId: {
		type: Sequelize.STRING,
		field: 'store_id' 
	},
	name: {
		type: Sequelize.STRING
	},
	address: {
		type: Sequelize.STRING
	},
	provinceId: {
		type: Sequelize.STRING,
		field: 'province_id' 
	},
	cityId: {
		type: Sequelize.STRING,
		field: 'city_id' 
	}
}, {
	freezeTableName: true,
	createdAt: false,
	updatedAt: false
});

module.exports = sequelize;
```

## 写法一

​	你可能会奇怪为什么会有两种写法，因为版本迭代的比较快，而每次迭代之后都会有一些比较大变化(这有可能是Facebook的一贯风格...)，所以市面上会有常见的两种写法，这种写法是稍微老一点的文章都是这样写的，当然这也是官方API支持的，所以不必担心。

**schema.js**

```javascript
var graphql = require('graphql');
var db = require('./db');
// 返回数据的描述，类似于java中的VO
var StoreType = new graphql.GraphQLObjectType({
	name: 'Store',
	fields: () => ({
		id: {
			type: graphql.GraphQLString
		},
		name: {
			type: graphql.GraphQLString
		},
		cityId: {
			type: graphql.GraphQLString
		},
		provinceId: {
			type: graphql.GraphQLString
		},
		address: {
			type: graphql.GraphQLString
		}
	})
});

var CityType = new graphql.GraphQLObjectType({
	name: 'City',
	fields: () => ({
		id: {
			type: graphql.GraphQLString
		},
		name: {
			type: graphql.GraphQLString
		},
		cityId: {
			type: graphql.GraphQLString
		},
		provinceId: {
			type: graphql.GraphQLString
		},
		cityList: {
			type: new graphql.GraphQLList(CityType),
			resolve(city) {
				// console.log('get Citys');
				return db.models.city.findAll({where: { provinceId: city.provinceId }});
			}
		},
		storeList: {
			type: new graphql.GraphQLList(StoreType),
			resolve(city) {
				return db.models.store.findAll({where: { provinceId: city.provinceId }});
			}
		}
	})
});
// 查询接口
const Query = new graphql.GraphQLObjectType({
	name: 'Query',
	fields: () => ({
		getCityList: {
			type: new graphql.GraphQLList(CityType),
			args: {
				id: {
					type: graphql.GraphQLString
				}
			},
			resolve(root, args) {
				if(args && args.id) {
					return db.models.city.findAll({where: args});
				}
				return db.models.city.findAll({where: { cityId: '' }});
			}
		}
	})
});
// 修改接口
const Mutation = new graphql.GraphQLObjectType({
	name: 'Mutation',
	fields: () => ({
		createCity: {
			type: CityType,
			args: {
				id: {
					type: new graphql.GraphQLNonNull(graphql.GraphQLString)
				},
				name: {
					type: new graphql.GraphQLNonNull(graphql.GraphQLString)
				},
				cityId: {
					type: new graphql.GraphQLNonNull(graphql.GraphQLString)
				},
				provinceId: {
					type: new graphql.GraphQLNonNull(graphql.GraphQLString)
				}
			},
			resolve(root, args) {
				console.log(args);
				return db.models.city.create(args);
			}
		}
	})
});
// Schema
const Schema = new graphql.GraphQLSchema({
	query: Query,
	mutation: Mutation
});

// 查询请求字符串
var queryStr = `
	query { 
		getCityList(id: "b06b7f27aa3138b96c389607d8225cc1") { 
			id
			provinceId
			name 
			cityList {
				#id
				name
			}
			storeList {
				name
				address
			}
		} 
	}
`;
// 修改请求字符串
var mutationStr = `
	mutation { 
		createCity(id: "123", name: "长白山", cityId: "001", provinceId: "001") { 
			id
		} 
	}
`;

var root = { };
// 执行
graphql.graphql(Schema, queryStr, root).then((response) => {
	console.log(JSON.stringify(response));
});
```

​		`GraphQLObjectType` 是返回值描述，相当于`Java`的VO，唯一不同的是在返回的字段中有一个`resolve`方法，此方法是为了获取该字段数据时所调用的方法，这里查询了数据库补全数据，下面`Query`和`Mutation`还增加了`args`是接收`query`的参数使用，可以理解成`Query`和`Mutation`中都是API中的方法，可以接受参数，而`type`就是返回值类型。

​	 `GraphQLSchema`是请求的入口，只有`query`和`mutation`，**这里有点奇怪，是不是我研究的不对，如果有人能愿意告诉我，感激不尽**。

​	再下面就是请求的字符串了，在请求中我们拼接了一个`getCityList`并且带了`id`作为参数，而后面括号里的内容则是需要返回的字段，`createCity`也是一样，只不过这个方法是修改，而不是查询，我们可以通过`GraphQLNonNull`来验证非空。

​	`#`在这里是注释。

返回值：

```json
{
    "data": {
        "getCityList": [
            {
                "id": "b06b7f27aa3138b96c389607d8225cc1", 
                "provinceId": "310000", 
                "name": "上海市", 
                "cityList": [
                    {
                        "name": "上海市"
                    }, 
                    {
                        "name": "青岛市"
                    }, 
                    {
                        "name": "苏州市"
                    }
                ], 
                "storeList": [
                    {
                        "name": "五角场店", 
                        "address": "上海市杨浦区邯郸路585号苏宁电器5楼"
                    }
                ]
            }
        ]
    }
}
```

Graphql的返回值如果成功一定会带上`data`，如果有错误则会多一个`errors`，如果验证都没有过就不会带`data`了，就像这样：

```json
{
    "data": {
        "storeList": null
    }, 
    "errors": [
        {
            "message": "获取门店列表错误！！！", 
            "locations": [
                {
                    "line": 3, 
                    "column": 3
                }
            ], 
            "path": [
                "storeList"
            ]
        }
    ]
}
```

## 写法二

​	看了**【写法一】**大家对Graphql一定有一些概念了，只是描述返回值有些繁琐，是不是有更简便的方式呢，这就是下面要讲的内容。

**schema.gql**

```gql
# 城市
type City {
	id: String
	provinceId: String
	cityId: String
	name: String
	cityList: [City]
	storeList: [Store]
}

# 门店
type Store {
	id: String
	provinceId: String
	cityId: String
	name: String
	address: String
}

# 查询
type Query {
	getCityList(id: String): [City]
	storeList(id: String): [Store]
}

# 修改
type Mutation {
	createCity (
		id: String!
		name: String!
		cityId: String!
		provinceId: String!
	): City
}
```

这是一个纯文本文件，通过IDE插件可以使语法高亮，还是刚刚的描述，是不是简洁多了呢，对于数据类型大家都能看到`[]`表示数组，`!`表示非空，我们再来看看js的代码。

**schema.js**

```javascript
var graphql = require('graphql');
var tools = require('graphql-tools');
var db = require('./db');
var fs = require('fs');
// 读取文本文件
var schemaStr = fs.readFileSync('./schema.gql', 'utf8');

var resolvers = {
	Query: {
		getCityList(root, args) {
			if(args && args.id) {
				return db.models.city.findAll({ where: args});
			}
			return db.models.city.findAll({where: { cityId: '' }});
		},
		storeList(root, args) {
			// if(args && args.id) {
			// 	return db.models.store.findAll({ where: args});
			// }
			// return db.models.store.findAll();
			throw new Error('获取门店列表错误！！！');
		}
	},
	Mutation: {
		createCity(root, args) {
			return db.models.city.create(args);
		}
	},
	City: {
		cityList(city) {
			return db.models.city.findAll({where: { provinceId: city.provinceId }});
		},
		storeList(city) {
			return db.models.store.findAll({where: { provinceId: city.provinceId }});
		}
	}
};

// 参考：http://dev.apollodata.com/tools/graphql-tools/generate-schema.html#api
var schema = tools.makeExecutableSchema({
	typeDefs: schemaStr,
	resolvers: resolvers,
});

var queryStr = `
	query { 
		getCityList { 
			id
			provinceId
			name 
			cityList {
				#id
				name
			}
			storeList {
				name
				address
			}
		} 
	}
`;

var queryStr2 = `
	query { 
		storeList(id: "015a92a2a82e763d50ce4bd8daa9770e") { 
			name
			address
		} 
	}
`;


var mutationStr = `
	mutation { 
		createCity(id: "345", name: "长白山", cityId: "001", provinceId: "001") { 
			id
		} 
	}
`;

var root = { };
graphql.graphql(schema, queryStr2, root).then((response) => {
	console.log(JSON.stringify(response));
});
```

​	还记得刚刚的`getCityList`和`createCity`方法吗？我们将他们挂在了`resolvers`对象上，VO上的`resolve`方法也挂在了它上面，通过[apollo](http://dev.apollodata.com/)的[graphql-tools](https://www.npmjs.com/package/graphql-tools)来帮助我们生成schema对象，官网也提供了`buildSchema`的方法，只不过太弱，不提也罢。

## Introspection([自省](http://graphql.org/learn/introspection/))

这是最让人激动的一个功能，当我们做了大量的开发，还有去自己手动补充文档真是无比痛苦，随着业务的调整文档永远跟不上代码，而Graphql帮我们很好的解决了这样的痛苦，我们将Query字符串换成：

```gql
{
  __schema {
    types {
      name
    }
  }
}
```

结果：

```json
{
    "data": {
        "__schema": {
            "types": [
                {
                    "name": "Query"
                }, 
                {
                    "name": "String"
                }, 
                {
                    "name": "City"
                }, 
                {
                    "name": "Store"
                }, 
                {
                    "name": "Mutation"
                }, 
                {
                    "name": "__Schema"
                }, 
                {
                    "name": "__Type"
                }, 
                {
                    "name": "__TypeKind"
                }, 
                {
                    "name": "Boolean"
                }, 
                {
                    "name": "__Field"
                }, 
                {
                    "name": "__InputValue"
                }, 
                {
                    "name": "__EnumValue"
                }, 
                {
                    "name": "__Directive"
                }, 
                {
                    "name": "__DirectiveLocation"
                }
            ]
        }
    }
}
```

能够看到我们写的所有描述的名字和内置的名字，也就是说我们可以通过内置的API来生成一份最新的文档，这里我汇总了一些内置的字段:

```gql
{
	__schema {
		queryType {
			name
			fields {
				name
				args {
					name
					type {
						name
						kind
						ofType {
							name
							kind
						}
					}
				}
				type {
					name
					kind
				}
			}
		}
		mutationType {
			name
			fields {
				name
				args {
					name
					type {
						name
						kind
						ofType {
							name
							kind
						}
					}
				}
				type {
					name
					kind
					ofType {
						name
						kind
					}
				}
			}
		}
		types {
			name
			fields {
				name
				type {
					name
					kind
				}
			}
		}
	}
	__type(name: "City") {
		name
		fields {
			name
			type {
				name
				kind
			}
		}
	}
}
```

## 总结

​	由于篇幅有限还有很多特性没有讲，大家有兴趣可以看[官方文档](http://graphql.org/learn/)或者[Apollo](http://dev.apollodata.com/)。

​	总的来说GraphQL还是有很多优点，可以帮助我们解决手机和PC请求数据适配问题，对增加、修改返回值能解决大部分的问题，好处不说了，说说缺点吧。

​	**缺点：**

1. 查询语句并不是js对象，只能通过字符串拼接，可能会造成错误。
2. 如果返回的字段特别多，客户端则需要拼接很多字段，当然[Fragments](http://graphql.org/learn/queries/#fragments) 可以解决这个问题，只是看起来很怪。
3. 返回的格式不能自定义，错误消息需要二次处理，GraphQL可以帮我们做一些非空验证，但是message都是英文的，并不能自定义，需要我们二次处理。



​	晚上照顾孩子没怎么睡好，可能写的有点乱，感谢您的阅读，如果能转载更是感激不尽，如有错误欢迎指正！