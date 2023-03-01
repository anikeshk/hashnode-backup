# Breaking Changes 🤝 Breaking Projects

Any codebase, from a small pet project to a large enterprise grade application, has some reliance on **third party dependencies or modules**. This could be to easily allow read/write operations on a database (like [elasticsearch](https://www.npmjs.com/package/@elastic/elasticsearch)), making operations on data structures easier (like [lodash](https://www.npmjs.com/package/lodash)), or to enable extensive testing capability (like [mocha](https://www.npmjs.com/package/mocha)). This lifts a lot of the heavy work one would need to do if they would have to natively build these functionalities themselves. But the one drawback of this is you are are reliant on the **maintainer** of the dependency to preserve the library and ensure compatibility as the core underlying language grows. I hope the lessons I learnt while working on a Node.js + dependency uplift at my company will be useful in your projects.

The below points and examples are presented in Node.js but can be applied to any language.

## Read The Release Notes

The first point is very obvious but has to be said. When a dependency is updated with breaking changes, the maintainer will publish **release notes** which contain a list of these changes along with all the other updates. You should understand these breaking changes and check if these impact your codebase in any way. Maintain a list of dependencies your project uses along with a link to the release notes page for easy accessibility.

Example of release notes for [mocha](https://github.com/mochajs/mocha/blob/master/CHANGELOG.md) which contains different types of changes from "Breaking Changes" to "Bug Fixes":

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650108955954/TJobO85h8.png align="left")

Innocuous updates like changing the library function name from `eval` to `evaluate` in [mathjs's 6.0.0 update](https://mathjs.org/history.html#20190608-version-600) can be easily missed if one does not carefully look at the release notes.

Also, some libraries will provide **migration guides** to help you transition your codebase from an older version to the newer - like this one by [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/7.17/breaking-changes.html) when Elastic completely revamped their Node.js client.

## Strict Dependency Version Management

Most programming languages allow you to define a list of third party dependencies along with a version. For Node.js you would use a `package.json`, for Python you would use a `requirements.txt`, and so on. Tools like [Dependabot](https://github.com/dependabot) have made dependency management easier, but you need to make sure you have **strict policies** in place on when you update the versions of dependencies. Blindly updating dependency versions without extensive testing is a recipe for disaster. Multiple stakeholders in the project must review the change, understand the impact and then only approve the update. The codebase also needs to be extensively tested to ensure all functionality that is either directly or indirectly impacted by this version update is working as expected.

Further, in the `package.json` the dependency version can be defined in three different ways to control [which semantic version is installed](https://docs.npmjs.com/about-semantic-versioning):

1. `"aerospike": "3.16.0"` - Install only **this specific version** (3.16.0)
    
2. `"aerospike": "~3.16.0"` - Install any **patch versions** from 3.16.0 (3.16.0 to &gt;3.17.0)
    
3. `"aerospike": "^3.16.0"` - Install any **minor versions** from 3.16.0 (3.16.0 to &gt;4.0.0)
    

You should choose which would be the most appropriate for your project - the first one being the most "safest" since neither newer patch or minor versions are installed.

There are still a few nuances in each definition, which are covered extensively in this [excellent video](https://www.youtube.com/watch?v=7lYnzRkVVLE) by Hussein Nasser.

## Test Functionality Reliant On Dependencies

You should ensure that dependencies are getting exercised and tested in your range of tests as well. Typically calls to databases are "stubbed" or "mocked" in unit tests, and hence the actual function call to the database as well as the result of the database operation is not tested. These tests should be written separately or carried out manually before updating a dependency version.

Here's an example where a ["bug fix"](https://github.com/aerospike/aerospike-client-nodejs/blob/master/CHANGELOG.md#3163---2021-02-09) in the Node.js Aerospike library introduced supported for the `boolean` values in their Map datatype, which completely changed how data is stored in the database - even with NO change in the codebase which uses this library.

You can find out how you can run this experiment on Docker by going to [github.com/anikeshk/aerospike-boolean-example](https://github.com/anikeshk/aerospike-boolean-example). This repository contains the code we are going to run in the below example.

Let's say you had a webpage which inputted an user's name, age, gender and then the user needed to choose dishes they liked from three choices. You then save these details and choices in an Aerospike database. Another backend service reads this data from Aerospike and does some processing on the user's choices.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650206688865/gkaqD4Fw6.png align="left")

For this example I have taken the example of an user "Tom" who likes burgers and tacos, and combined both write and read operation into one file for simplicity. Let's use two different versions of the aerospike npm module and check the results.

Initially, if we use the ["aerospike" npm module](https://www.npmjs.com/package/aerospike) with version `3.11.0`, this would be the output after running the program:

```plaintext
Connection to Aerospike cluster succeeded!
User data stored!
User data fetched! {"age":21,"gender":"male","food":{"pizza":0,"burger":1,"taco":1}}
```

Aerospike stores these choices in a `Map` and converts `true` or `false` to 1 or 0 respectively. You can see this reflected in the above output.

And if we check the record in the Aerospike database using `aql`, we can see even the database stores the choices as 0 or 1. While processing the data we need to assume 0 means `false` and 1 means `true.`

```plaintext
aql> select * from test.users where PK="tom"
+-------+-----+--------+------------------------------------------+
| PK    | age | gender | food                                     |
+-------+-----+--------+------------------------------------------+
| "tom" | 21  | "male" | MAP('{"pizza":0, "burger":1, "taco":1}') |
+-------+-----+--------+------------------------------------------+
1 row in set (0.001 secs)
```

Now, let's use the aerospike npm module with version `3.16.3` instead. On running the program with `3.16.3` here is the output:

```plaintext
Connection to Aerospike cluster succeeded!
User data stored!
User data fetched! {"age":21,"gender":"male","food":{"pizza":false,"burger":true,"taco":true}}
```

We get `true` or `false` instead of 1 or 0! With this minor version update, Aerospike has changed how it stores boolean values in the database inside a `Map`, even though we didn't change our own code. A change in the dependency has forced a change in our codebase's data processing. If we didn't have tests to verify the datatype of the choices in our database after the write operation, we could impact all our downstream processors with this one minor version change.

This concludes the different techniques you can use to ensure breaking changes in dependencies do not break your own projects!

Here's quick summary of the points discussed:

1. Read the release notes of the dependency you plan to update to check for any breaking changes
    
2. Have strict version update policies in place in your team or organization - do not auto-approve even minor version updates without a review
    
3. Test out all the functionality which is reliant on the dependencies so can you can catch changes which break your own codebase easily