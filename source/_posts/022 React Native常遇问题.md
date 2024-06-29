---
title: React Native常遇问题
excerpt: 在使用expo开发过程中经常会遇到一些问题，这里记录一下
published: true
time: "2024/06/29 20:20:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/1719662257028.png
category: 
- [编程,前端,ReactNative]
tags:
  - 编程
  - React Native
  - 问题
  - 软件开发
---


## 1、`TextInput`聚焦弹起键盘

使用以下方式并不能直接弹起键盘，原因尚不清楚，可能是由于`setModalVisible`的异步性导致的

```tsx
const toggleModal = async () => {
    await setModalVisible(!isModalVisible);
    if(!isModalVisible){
      inputRef?.current?.focus();
    }
};
```

可以变化一下思路，聚焦——>失去焦点——>聚焦，聚焦10ms之后失去焦点，失去焦点触发`onBlur`函数后再聚焦

代码如下

```tsx
const toggleModal = async () => {
    await setModalVisible(!isModalVisible);
    console.log(inputRef);
    if(!isModalVisible){
      inputRef?.current?.focus();
      setTimeout(() => {
        Keyboard.dismiss()
      }, 10);
    }
};

// 输入框
<TextInputonBlur onBlur={()=>{inputRef?.current?.focus()}}/>
```

> 这里一定要使用`async`和`await`

## 2、`useRef`获取组件实例

使用钩子`useRef`获取组件

```tsx
// 创建useRef实例
const textInput:any = useRef(null);

// 挂在
<TextInput
    ref={textInput}
    value={searchText}
    onChangeText={text => setSearchText(text)}
  />

// 获取组件实例
textInput?.current?.value();
```

## 3、父组件调用子组件方法并传值

1. 父子间挂在子组件

```tsx
const notionModalRef:any = useRef(null)

<CreateNotionModal props={{toggleModal,isModalVisible,id}} ref={notionModalRef}/>
```

其中`props`用于传递对象值，`ref`用于传递组件

2. 获取组件以及数据

```tsx
export default forwardRef(({props}:any,ref:any) => {
    const {notionModalRef} = ref
    const {toggleModal,isModalVisible,id} = props

    // 子组件向父组件暴露方法
    useImperativeHandle(notionModalRef, () => ({
        inputOnFocus
      }));

    // 切换模态框
    const inputOnFocus = () => {
   
    };
    
    return (
        <View style={styles.centeredView}>
            <View style={styles.modalView}>
                <TextInput
                multiline={true}
                style={styles.input}
                maxLength={1000}
                autoFocus={true}
                value={textInput}
                ref={inputRef}
                onBlur={()=>{inputRef?.current?.focus()}}
                onChangeText={text => setTextInput(text)}/>

                <View style={styles.modalBtn}>
                <Pressable onPress={()=>{toggleModal()} }>
                {/* <Pressable onPress={()=>{} }> */}
                    {({ pressed }) => (
                    <Text style={styles.cancel}>取消</Text>
                    )}
                </Pressable>
                <Pressable onPress={async()=>{setTextInput(await pasteFromClipboard(textInput))} }>
                    {({ pressed }) => (
                    <Ionicons
                        color={Colors.light.tint}
                        name="clipboard-outline"
                        size={30}
                        style={{ marginRight: 15, opacity: pressed ? 0.5 : 1 }}
                    />
                    )}
                </Pressable>
                <Pressable onPress={create }>
                    {({ pressed }) => (
                    <Ionicons
                        color={Colors.light.tint}
                        name="send-outline"
                        size={30}
                        style={{ marginRight: 15, opacity: pressed ? 0.5 : 1 }}
                    />
                    )}
                </Pressable>
                </View>
            </View>
        </View>
    )
})
```

3. 使用

```tsx
notionModalRef?.current?.inputOnFocus()
```

> 我这里用到了`useState`的数据，注意状态变化，即组件是否显示与`notionModalRef?.current?.inputOnFocus()`的关系

## 4、expo抽屉打开与关闭

试了很多种方法，都是显示`closeDrawer`未定义

可以使用`@react-navigation/native`中的`DrawerActions`和`useNavigation`组合实现抽屉打开与关闭

```tsx
import { DrawerActions,useNavigation } from '@react-navigation/native';

const CustomDrawer = () => {
	const navigation = useNavigation();
    const tagRename = () => {
        navigation.dispatch(DrawerActions.closeDrawer())
    }
}
```

## 5、全局数据储存

使用`useContext`存储全局变量。

1. 首先创建`context`实例，对外暴露一个数据对象，里面可包含数值、函数等

```tsx
type SqliteType = {
    db:SQLite.SQLiteDatabase | null | any;
    openDb:()=>void;
    getDbFile:() => void;
    exeSql:(type:string,data:any[]) => Promise<any>;
}

const SqliteContext = createContext<SqliteType>({
    db:null,
    openDb:()=>{},
    getDbFile:()=>{},
    exeSql:async()=>{},
})
```

2. 创建提供者

> 这里使用`useRef`创建数据，使用`useState`在异步函数中会出现问题

```tsx
const SqliteProvider = ({children}:PropsWithChildren) => {
    const db = useRef<null|SQLite.SQLiteDatabase>(null)

    // 打开数据库
    const openDb = () => {
        
    }

    // 获取远程db数据
    const getDbFile = async () => {
        
    }

    // 执行语句
    const exeSql = async (type:string,data:any[]) => {
        
      }

    ...
}
```

3. 对外暴露的数据

```tsx
const SqliteProvider = ({children}:PropsWithChildren) => {
    ...

    return (
        <SqliteContext.Provider value={{db,openDb,getDbFile,exeSql}}>
            {children}
        </SqliteContext.Provider>
    )
}
```

4. 导出提供者及`useContext(SqliteContext)`实例

```tsx
export default SqliteProvider

export const useSqlite = () => useContext(SqliteContext)
```

5. 监听组件

```tsx
function RootLayoutNav() {
  return (
	<SqliteProvider>
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(drawer)" options={{ headerShown: false }} />
      </Stack>
    </SqliteProvider>
  );
}

```

6. 使用

```tsx
const {exeSql,getDbFile} = useSqlite()
```

## 6、`sqlite`实现增删改查

这里使用`execAsync`异步函数获取数据

```tsx
// 执行语句
const exeSql = async (type:string,data:any[]) => {
    try {
        if(db.current==null){
            console.log("数据库不存在");
            openDb()
        }
        const readOnly = false;
        return db?.current?.execAsync([{ sql: sqls[type],args: data }],readOnly).then((result:any)=>{
            const data = result[0]?.rows
            console.log("执行结果",result[0]?.rows);
            return data
        })
      } catch (error) {
        console.error('An error occurred:', error);
        throw error;
      }
  }
```

使用

```tsx
exeSql("searchAllTags",[]).then((res)=>{
 console.log(res)
})
```

> 这里封装一些`sql`语句

```tsx
const sqls:any = {
    "searchAllNotions":"SELECT * FROM NOTIONS",
    "insertNotion":"INSERT INTO NOTIONS (id,content,tag,create_time,update_time) VALUES (?, ?, ?, ?,?)",
    "updateNotionById":"UPDATE notions SET content = ?,update_time = ? WHERE id = ?",
    "searchNotionById":"SELECT content FROM notions WHERE id = ?",
    "searchTagNameById":"SELECT name FROM tags WHERE id = ?",
    "searchTagIdByName":"SELECT id FROM tags WHERE name = ?",
    "searchChildrenTagsById":"SELECT name FROM tags WHERE father = ?",
    "searchAllTags":"SELECT * FROM TAGS"
}
```

## 7、`dayjs`简单使用

导入库

```tsx
import dayjs from 'dayjs';
```

生成时间（毫秒）

```tsx
dayjs().valueOf()
```

生成时间

```tsx
dayjs().subtract(1, 'hour').toISOString()
```

计算开始到现在的天数

```tsx
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';

dayjs.extend(relativeTime);

console.log(dayjs(created_at).fromNow())
```

## 8、获取页面管线列表

当前页面处于的管线位置

```tsx
const segments = useSegments()

> ["(user)", "film"]
```

## 9、vscode配置prettier

[prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

安装依赖

```bash
npm i -D prettier eslint-config-prettier eslint-plugin-prettier
```

修改vscode设置

```json
"editor.defaultFormatter": "esbenp.prettier-vscode",
"files.autoSave": "onFocusChange",
"editor.formatOnSave": true, // 保存时自动格式化
"editor.formatOnType": true,
"[typescript]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"prettier.useEditorConfig": false // 一定要取消，否则会使用vscode工作区设置，而不会使用.prettierrc.js
```

在页面下新增文件`.prettierrc.js`，以下是我的默认配置

```js
/**
 * @see https://prettier.io/docs/en/options.html#print-width
 * @author lcm
 */
module.exports = {
  /**
   * 换行宽度，当代码宽度达到多少时换行
   */
  printWidth: 80,
  /**
   * 缩进的空格数量
   * @default 2
   * @type {number}
   */
  tabWidth: 2,
  /**
   * 是否使用制表符代替空格
   * @default false
   * @type {boolean}
   */
  useTabs: false,
  /**
   * 是否在代码块结尾加上分号
   * @default true
   * @type {boolean}
   */
  semi: true,
  /**
   * 是否使用单引号替代双引号
   * @default false
   * @type {boolean}
   */
  singleQuote: false,
  /**
   * 对象属性的引号处理
   * @default "as-needed"
   * @type {"as-needed"|"consistent"|"preserve"}
   */
  quoteProps: 'as-needed',
  /**
   * jsx中是否使用单引号替代双引号
   * @default false
   * @type {boolean}
   */
  jsxSingleQuote: false,
  /**
   * 末尾是否加上逗号
   * @default "es5"
   * @type {"es5"|"none"|"all"}
   */
  trailingComma: 'all',
  /**
   * 在对象，数组括号与文字之间加空格 "{ foo: bar }"
   * @default true
   * @type {boolean}
   */
  bracketSpacing: true,
  /**
   * 把多行HTML (HTML, JSX, Vue, Angular)元素的>放在最后一行的末尾，而不是单独放在下一行(不适用于自关闭元素)。
   * @default false
   * @type {boolean}
   */
  bracketSameLine: false,
  /**
   * 当箭头函数只有一个参数是否加括号
   * @default "always"
   * @type {"always"|"avoid"}
   */
  arrowParens: 'avoid',
  /**
   * 为HTML、Vue、Angular和Handlebars指定全局空格敏感性
   * @default "css"
   * @type {"css"|"strict"|"ignore"}
   */
  htmlWhitespaceSensitivity: 'ignore',
  /**
   * 是否缩进Vue文件中的<script>和<style>标记内的代码。有些人(比如Vue的创建者)不使用缩进来保存缩进级别，但这可能会破坏编辑器中的代码折叠。
   * @default "always"
   * @type {"always"|"avoid"}
   */
  vueIndentScriptAndStyle: false,
  /**
   * 文件结束符
   * @default "lf"
   * @type {"lf"|"crlf"|"cr"|"auto"}
   */
  endOfLine: 'crlf',
  /**
   * 因为使用了一些折行敏感型的渲染器（如GitHub comment）而按照markdown文本样式进行折行
   */
  proseWrap: 'never',
  // 是否使用根目录下的EditorConfig配置文件
  useEditorConfig: false,
  /**
   * HTML\VUE\JSX每行只有单个属性
   * @default true
   * @type {boolean}
   */
  singleAttributePerLine: true,
  disableLanguages: ['html'],
}
```

> 每次修改`.prettierrc.js`重启才能生效

## 10、解决异步获取数据组件不刷新

可以新增一个状态，数据加载时为`false`，加载成功后变为`true`

```ts
  const getData = async () => {
    setIsLodingData(true); // 设置加载状态为 true
    const notionsPromise = exeSql("searchAllNotions", []).then(async res => {
      for (let i = 0; i < res.length; i++) {
        await exeSql("searchTagNameById", [res[i].tag]).then(res2 => {
          res[i].tag = res2[0].name;
        });
      }
      return res;
    });

    const tagsPromise = exeSql("searchAllTags", []);

    const [notions, tags] = await Promise.all([notionsPromise, tagsPromise]);

    setAllNotions(notions);
    setTags(tags);
    setIsLodingData(false);
  };

{/* 数据展示卡片 */}
  {isLodingData ? (
	<ActivityIndicator
	  animating={isLodingData}
	  color={Colors.light.tint}
	/>
  ) : (
	<FlatList
	  data={notions.current}
	  renderItem={({ item }) => (
		<CartItem
		  cartType="show"
		  notion={item}
		  func={getData}
		/>
	  )}
	  contentContainerStyle={{
		gap: defalutSize,
		padding: defalutSize * 0.5,
	  }}
	/>
  )}
```

> `getData`函数中有两个异步加载函数，为了提高获取数据的效率，可以同时异步加载数据，加载完成后再赋值

之前的代码为

```ts
  const getData = async () => {
    await exeSql("searchAllNotions", []).then(async res => {
      for (let i = 0; i < res.length; i++) {
        await exeSql("searchTagNameById",[res[i].tag]).then((res2)=>{
          res[i].tag = res2[0].name
        })
      }
      setAllNotions(res);
    })
	
    });
    await exeSql("searchAllTags", []).then(async res => {
      setTags(res);
    });
  };
```
## 11、useEffect和useRef区别

### 11.1 定义

1. `useEffect`：管理状态。管理函数组件的状态和更新状态，变化后重新渲染徐建。
2. `useRef`：操作DOM元素。用于函数式组件中访问的全局变量，而不必渲染组件。

## 12、返回页面刷新

使用页面监听事件，监听`navigate`的值，一旦返回页面，`navigate`的值就会发生改变，从而触发刷新函数。

```tsx
import { useNavigation } from "expo-router";

const navigation = useNavigation();

// 返回页面刷新
useEffect(() => {
	const unsubscribe = navigation.addListener("focus", () => {
	  onRefresh();
	});
	return unsubscribe;
}, [navigation]);
```

## 13、Provider顺序问题

在写简记软件时，将数据库操作和灵感都封装成了全局`provider`，在灵感里使用到数据库的相关操作。

```tsx
// DataProvider

const updateNotion = async (textInput: string, id: string) => {
    await exeSql("searchNotionById", [id]).then(
      async (searchNotionByIdRes: any) => {
        if (searchNotionByIdRes[0]?.rows[0]?.content === textInput) {
          return;
        }
        const updata_time = dayjs().valueOf();
        await exeSql("updateNotionById", [textInput, updata_time, id]).then(
          () => {},
        );
      },
    );
  };
```

在封装时需要将`DataProvider`放在`SqliteProvider`里面，不然在使用`DataProvider`时会找不到数据库，因为先加载的`DataProvider`时`SqliteProvider`还未加载。