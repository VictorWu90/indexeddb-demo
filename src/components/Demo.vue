<template>
    <div>
        <Tabs value="name1">
            <TabPane label="建立连接" name="name1">
                <div style="text-align:left">
                    <div>
                        <Button type="success" @click="doSetup"
                            >建立连接</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                    <div>
                        <span>数据库名：{{ dbInfo.name }}</span>
                        <br />
                        <span>版本号：{{ dbInfo.version }}</span>
                    </div>
                </div>
            </TabPane>
            <TabPane label="创建表" name="name2">
                <div style="text-align:left">
                    <div>
                        <Button type="success" @click="doCreateTable"
                            >创建表</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                    <div>
                        <span>数据库名：{{ dbInfo.name }}</span>
                        <br />
                        <span>版本号：{{ dbInfo.version }}</span>
                    </div>
                </div>
            </TabPane>
            <TabPane label="添加数据" name="name3">
                <div style="text-align:left">
                    <div>
                        <Button type="success" @click="doAddData"
                            >添加数据</Button
                        >
                        <Button
                            type="success"
                            @click="doAddMultData"
                            style="margin-left:30px"
                            >添加批量数据</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                </div>
            </TabPane>
            <TabPane label="添加索引查询数据" name="name4">
                <div style="text-align:left">
                    <div>
                        <Button type="success" @click="doAddIndex"
                            >添加索引</Button
                        >
                        <Button
                            type="success"
                            @click="doSearchDataFromIndex"
                            style="margin-left:30px"
                            >查询数据</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                </div>
            </TabPane>
            <TabPane label="事务处理" name="name5">
                <div style="text-align:left">
                    <div>
                        <Button type="success" @click="doTransaction"
                            >事务示例</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                </div>
            </TabPane>
            <TabPane label="查询" name="name6">
                <div style="text-align:left">
                    <div>
                        <Button
                            type="success"
                            @click="doSearchDataFromIndexWithLeftLike"
                            >模糊查询左like</Button
                        >
                    </div>
                    <div>
                        <Button
                            type="success"
                            @click="doSearchDataFromIndexWithLike"
                            >模糊查询包含</Button
                        >
                    </div>
                    <div>
                        <Button
                            type="success"
                            @click="doSearchDataFromIndexWithCompoundQuery"
                            >复合查询</Button
                        >
                    </div>
                    <div>
                        <span>运行结果：{{ msg }}</span>
                    </div>
                </div>
            </TabPane>
        </Tabs>
    </div>
</template>

<script>
let db;
export default {
    data() {
        return {
            msg: "",
            dbInfo: {
                name: "",
                version: ""
            }
        };
    },
    mounted() {},
    methods: {
        check() {
            return new Promise((resolve, reject) => {
                if (!window.indexedDB) {
                    reject(new Error("该浏览器不支持indexedDB"));
                } else {
                    resolve();
                }
            });
        },
        open() {
            return new Promise((resolve, reject) => {
                let request = window.indexedDB.open("SchoolDatabase");
                request.onerror = e => {
                    reject(
                        new Error(
                            "创建数据库连接失败: " + e.target.error.message
                        )
                    );
                };
                request.onsuccess = e => {
                    // 创建数据库连接成功
                    // 获得IDBDatabase对象
                    let db = e.target.result;
                    resolve(db);
                };
            });
        },
        openAndCreateTable() {
            return new Promise((resolve, reject) => {
                let request = window.indexedDB.open(
                    "SchoolDatabase",
                    this.dbInfo.version + 1
                );
                request.onerror = e => {
                    reject(new Error("创建表失败: " + e.target.error.message));
                };
                request.onsuccess = e => {
                    // 创建数据库连接成功
                    // 获得IDBDatabase对象
                    let db = e.target.result;
                    resolve(db);
                };
                request.onupgradeneeded = e => {
                    let db = e.target.result;
                    if (!Array.from(db.objectStoreNames).includes("student")) {
                        // 创建使用id作为主键的student表
                        db.createObjectStore("student", { keyPath: "id" });
                    }
                    if (!Array.from(db.objectStoreNames).includes("teacher")) {
                        // 创建自增主键的teacher表
                        db.createObjectStore("teacher", {
                            autoIncrement: true
                        });
                    }
                };
            });
        },
        openAndAddIndex() {
            return new Promise((resolve, reject) => {
                let request = window.indexedDB.open(
                    "SchoolDatabase",
                    this.dbInfo.version + 1
                );
                request.onerror = e => {
                    reject(
                        new Error("创建索引失败: " + e.target.error.message)
                    );
                };
                request.onsuccess = e => {
                    // 创建数据库连接成功
                    // 获得IDBDatabase对象
                    let db = e.target.result;
                    resolve(db);
                };
                request.onupgradeneeded = e => {
                    let store1 = e.target.transaction.objectStore("student");
                    let store2 = e.target.transaction.objectStore("teacher");
                    if (!Array.from(store1.indexNames).includes("ageIndex")) {
                        store1.createIndex("ageIndex", "age", {
                            unique: false
                        });
                    }
                    if (!Array.from(store1.indexNames).includes("classIndex")) {
                        store1.createIndex("classIndex", "class", {
                            unique: false
                        });
                    }
                    if (!Array.from(store1.indexNames).includes("age|class")) {
                        store1.createIndex("age|class", ["age", "class"], {
                            unique: false
                        });
                    }
                    if (!Array.from(store2.indexNames).includes("classIndex")) {
                        store2.createIndex("classIndex", "class", {
                            unique: true
                        });
                    }
                };
            });
        },
        close() {
            if (db) {
                db.close();
            }
            db = null;
        },
        async setup() {
            if (db) {
                this.dbInfo = {
                    name: db.name,
                    version: db.version
                };
            } else {
                await this.check();
                db = await this.open();
                this.dbInfo = {
                    name: db.name,
                    version: db.version
                };
            }
        },
        async createTable() {
            if (
                !Array.from(db.objectStoreNames).includes("student") ||
                !Array.from(db.objectStoreNames).includes("teacher")
            ) {
                this.close();
                db = await this.openAndCreateTable();
                this.dbInfo = {
                    name: db.name,
                    version: db.version
                };
            }
        },
        async createIndex() {
            let store1 = db
                .transaction("student", "readonly")
                .objectStore("student");
            let store2 = db
                .transaction("teacher", "readonly")
                .objectStore("teacher");
            if (
                !Array.from(store1.indexNames).includes("ageIndex") ||
                !Array.from(store1.indexNames).includes("classIndex") ||
                !Array.from(store1.indexNames).includes("age|class") ||
                !Array.from(store2.indexNames).includes("classIndex")
            ) {
                this.close();
                db = await this.openAndAddIndex();
                this.dbInfo = {
                    name: db.name,
                    version: db.version
                };
            }
        },
        async addData(data) {
            let addPromise = (objectStore, obj) => {
                return new Promise((resolve, reject) => {
                    let request = objectStore.add(obj);
                    request.onsuccess = () => {
                        resolve();
                    };
                    request.onerror = e => {
                        reject(
                            new Error("添加数据失败: " + e.target.error.message)
                        );
                    };
                });
            };
            let objs = Array.isArray(data) ? data : [data];
            let transaction1 = db.transaction("student", "readwrite");
            let transaction2 = db.transaction("teacher", "readwrite");
            return new Promise((resolve, reject) => {
                let objectStore1 = transaction1.objectStore("student");
                let objectStore2 = transaction2.objectStore("teacher");
                Promise.all(
                    objs
                        .filter(obj =>
                            ["teacher", "student"].includes(obj.role)
                        )
                        .map(obj =>
                            addPromise(
                                obj.role === "student"
                                    ? objectStore1
                                    : objectStore2,
                                obj
                            )
                        )
                )
                    .then(() => {
                        resolve();
                    })
                    .catch(e => {
                        transaction1.abort(e.message);
                        transaction2.abort(e.message);
                        reject(new Error(e.message));
                    });
            });
        },
        async searchStudentFromIndex() {
            return new Promise((resolve, reject) => {
                let transaction = db.transaction("student", "readonly");
                let objectStore = transaction.objectStore("student");
                let dbIndex = objectStore.index("ageIndex");
                let request = dbIndex.openCursor(IDBKeyRange.bound(0, 30));
                let result = [];
                request.onsuccess = e => {
                    let cursor = e.target.result;
                    if (cursor) {
                        result.push(cursor.value);
                        cursor.continue();
                    } else {
                        resolve(result);
                    }
                };
                request.onerror = e => {
                    reject(
                        new Error("搜索数据失败: " + e.target.error.message)
                    );
                };
            });
        },
        async searchTeacherFromIndex() {
            return new Promise((resolve, reject) => {
                let transaction = db.transaction("teacher", "readonly");
                let objectStore = transaction.objectStore("teacher");
                let dbIndex = objectStore.index("classIndex");
                let request = dbIndex.openCursor(IDBKeyRange.only("1-2"));
                let result = [];
                request.onsuccess = e => {
                    let cursor = e.target.result;
                    if (cursor) {
                        result.push(cursor.value);
                        cursor.continue();
                    } else {
                        resolve(result);
                    }
                };
                request.onerror = e => {
                    reject(
                        new Error("搜索数据失败: " + e.target.error.message)
                    );
                };
            });
        },
        async searchStudentFromIndexWithLeftLike() {
            return new Promise((resolve, reject) => {
                let transaction = db.transaction("student", "readonly");
                let objectStore = transaction.objectStore("student");
                let dbIndex = objectStore.index("classIndex");
                let request = dbIndex.openCursor(
                    IDBKeyRange.bound("1-", "1-" + "\uffff")
                );
                let result = [];
                request.onsuccess = e => {
                    let cursor = e.target.result;
                    if (cursor) {
                        result.push(cursor.value);
                        cursor.continue();
                    } else {
                        resolve(result);
                    }
                };
                request.onerror = e => {
                    reject(
                        new Error("搜索数据失败: " + e.target.error.message)
                    );
                };
            });
        },
        async searchStudentFromIndexWithLike() {
            return new Promise((resolve, reject) => {
                let transaction = db.transaction("student", "readonly");
                let objectStore = transaction.objectStore("student");
                let dbIndex = objectStore.index("classIndex");
                let request = dbIndex.openCursor();
                let result = [];
                request.onsuccess = e => {
                    let cursor = e.target.result;
                    if (cursor) {
                        if ((cursor.value.class || "").includes("2")) {
                            result.push(cursor.value);
                        }
                        cursor.continue();
                    } else {
                        resolve(result);
                    }
                };
                request.onerror = e => {
                    reject(
                        new Error("搜索数据失败: " + e.target.error.message)
                    );
                };
            });
        },
        async searchStudentFromIndexWithCompoundQuery() {
            return new Promise((resolve, reject) => {
                let transaction = db.transaction("student", "readonly");
                let objectStore = transaction.objectStore("student");
                let dbIndex = objectStore.index("age|class");
                let request = dbIndex.openCursor(
                    IDBKeyRange.bound([0, "1-1"], [20, "1-1"])
                );
                let result = [];
                request.onsuccess = e => {
                    let cursor = e.target.result;
                    if (cursor) {
                        result.push(cursor.value);
                        cursor.continue();
                    } else {
                        resolve(result);
                    }
                };
                request.onerror = e => {
                    reject(
                        new Error("搜索数据失败: " + e.target.error.message)
                    );
                };
            });
        },
        async doSetup() {
            try {
                this.setup();
                this.msg = "创建数据库连接成功";
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doCreateTable() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功";
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doAddData() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加数据";
                await this.addData([
                    {
                        id: "0001",
                        name: "Victor",
                        age: 29,
                        role: "student",
                        class: "1-1"
                    },
                    {
                        name: "Jack",
                        age: 42,
                        role: "teacher",
                        class: "1-1"
                    }
                ]);
                this.msg = "添加数据成功";
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doAddMultData() {
            let objs = [
                {
                    id: "0002",
                    name: "Lucy",
                    age: 15,
                    role: "student",
                    class: "1-1"
                },
                {
                    id: "0003",
                    name: "Mike",
                    age: 35,
                    role: "student",
                    class: "1-2"
                },
                {
                    id: "0004",
                    name: "David",
                    age: 28,
                    role: "student",
                    class: "1-2"
                },
                {
                    id: "0005",
                    name: "Jim",
                    age: 28,
                    role: "student",
                    class: "2-1"
                },
                {
                    id: "0006",
                    name: "Tom",
                    age: 28,
                    role: "student",
                    class: "2-2"
                },
                {
                    name: "John",
                    age: 40,
                    role: "teacher",
                    class: "1-2"
                }
            ];
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加数据";
                await this.addData(objs);
                this.msg = "添加数据成功";
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doAddIndex() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加索引";
                await this.createIndex();
                this.msg = "添加索引成功";
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doSearchDataFromIndex() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加索引";
                await this.createIndex();
                this.msg = "添加索引成功，正在查询数据";
                let students = await this.searchStudentFromIndex();
                this.msg =
                    "查询数据成功，年龄在0-30之间的学生为: " +
                    students.map(student => student.name).join("|");
                let teachers = await this.searchTeacherFromIndex();
                this.msg +=
                    "查询数据成功，班级在1-2的老师为: " +
                    teachers.map(teacher => teacher.name).join("|");
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doSearchDataFromIndexWithLeftLike() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加索引";
                await this.createIndex();
                this.msg = "添加索引成功，正在查询数据";
                let students = await this.searchStudentFromIndexWithLeftLike();
                this.msg +=
                    "查询数据成功，班级在1-?的学生为: " +
                    students.map(student => student.name).join("|");
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doSearchDataFromIndexWithLike() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加索引";
                await this.createIndex();
                this.msg = "添加索引成功，正在查询数据";
                let students = await this.searchStudentFromIndexWithLike();
                this.msg +=
                    "查询数据成功，班级中有2的学生为: " +
                    students.map(student => student.name).join("|");
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doSearchDataFromIndexWithCompoundQuery() {
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加索引";
                await this.createIndex();
                this.msg = "添加索引成功，正在查询数据";
                let students = await this.searchStudentFromIndexWithCompoundQuery();
                this.msg +=
                    "查询数据成功，在1-1班里年龄在0-20之间的学生为: " +
                    students.map(student => student.name).join("|");
            } catch (error) {
                this.msg = error.message;
            }
        },
        async doTransaction() {
            let objs = [
                {
                    id: "0007",
                    name: "Jay",
                    age: 32,
                    role: "student",
                    class: "1-2"
                },
                {
                    name: "Mar",
                    age: 39,
                    role: "teacher",
                    class: "1-1"
                },
                {
                    id: "0004",
                    name: "David",
                    age: 28,
                    class: "1-2",
                    role: "student"
                }
            ];
            try {
                await this.setup();
                this.msg = "数据库连接成功, 正在初始化表";
                await this.createTable();
                this.msg = "初始化表成功, 正在添加数据";
                await this.addData(objs);
                this.msg = "添加数据成功";
            } catch (error) {
                this.msg = error.message;
            }
        }
    }
};
</script>
<style lang="scss" scoped></style>
