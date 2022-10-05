---
id: supabase
title: Supabase
---
import login from '@site/static/img/guides-and-concepts/data-provider/supabase/login-screen.png';
import welcome from '@site/static/img/guides-and-concepts/data-provider/supabase/welcome.png';

Supabase is an open-source Firebase alternative that provides backend features. This tutorial steps will be focus specifically on database and authentication features.  We'll see how to use Supabase as a data provider and implement authentication to refine app.

refine offers built-in data provider support for Supabase and handles all required data service methods out-of-the-box. Therefore, we will not need to use complex boilerplately codes to make it work. refine handles all the hard works for us by internal hooks and implementations.

We'll build a simple CRUD app with refine and use Supabase as a data provider.  We'll also see how to use Supabase's authentication features on refine app.  

We are assuming that you have already knowledge about how refine works. If not, please check out the [Tutorial](https://refine.dev/docs/tutorials/ant-design-tutorial/) section.


[Refer to docs for more information about data provider &#8594](https://refine.dev/docs/api-reference/core/providers/data-provider/)

[Discover the +15 most popular backend service data providers supported out-of-the-box by refine &#8594](https://refine.dev/integrations/)

## Project Setup

We'll be using superplate CLI to bootstrap our example project with special preset defined to supabase example

```
npx superplate-cli --preset refine-antd-supabase my-supabase-app
```

Also we need to install npm packages to use markdown editor:

```
npm i react-markdown react-mde
```

This will create a new refine app with supabase data provider and Ant Desing as a UI framework. We'll be using this project as a base to implement our example.

You are free to bootstrap a refine app with any other features you want.  To do so, you can use run following command and choose any data provider or features you want.


```
npx superplate-cli example-app
```

:::tip
If you want to add Supabase data provider to existed refine app, you add it by running:

```
npm i @pankod/refine-supabase
```
:::



## Establishing Supabase connection

### Initialize Supabase client

If you head over to `src/utilty` folder, you'll see a file called `supabaseClient.ts` created by CLI. This auto-generated file contains API credentials and a function that initializes the Supabase client.  

```ts
import { createClient } from "@pankod/refine-supabase";

const SUPABASE_URL = "https://iwdfzvfqbtokqetmbmbp.supabase.co";
const SUPABASE_KEY =
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYW5vbiIsImlhdCI6MTYzMDU2NzAxMCwiZXhwIjoxOTQ2MTQzMDEwfQ._gr6kXGkQBi9BM9dx5vKaNKYj_DJN1xlkarprGpM_fU";

export const supabaseClient = createClient(SUPABASE_URL, SUPABASE_KEY);
```

We'll be using this example API credentials and `createClient` method that exposes from `refine-supabase` package for enabling refine to Supabase API connection.

[You can find your Supabase URL and key from your Supabase dashboard &#8594 ](https://supabase.com/docs/guides/with-react#get-the-api-keys)

:::tip
You can also use environment variables to store your Supabase URL and key. This is a good practice to keep your sensitive information safe.
:::

### Register Supabase data provider
Let's head over to `App.tsx` file where all magic happens. This is the entry point our app. We'll be registering our Supabase data provider here.

```tsx title="App.tsx"
import { Refine } from "@pankod/refine-core";
...
 // highlight-start
import { dataProvider } from "@pankod/refine-supabase";
import { supabaseClient } from "utility";
// highlight-end

function App() {
  return (
    <Refine
      ...
      // highlight-next-line
      dataProvider={dataProvider(supabaseClient)}
      ...
    />
  );
}

export default App;
```

Highlighted lines are the ones CLI generator automatically added to register Supabase data provider. Simply, we are passing `supabaseClient` to `dataProvider` method to establish connection with Supabase API.

With this configuration, refine can now communicate with Supabase API and perform all required data service CRUD methods using data hooks.

[Refer to documentation to learn more about how to use data hooks &#8594](https://refine.dev/docs/api-reference/core/hooks/data-hooks/)


## Understanding Auth Provider 

Auth provider is a concept that allows us to use any authentication service with refine. 

You'll see a file called `src/authProvider.ts` created by CLI. This auto-generated file contains functions that using Supabase Auth API methods to perform authentication and authorization operations.So basically, this is where we set authentication logic for the app.

[Refer to refine Auth Provider docs more information &#8594](https://refine.dev/docs/api-reference/core/providers/auth-provider/)


<details><summary>Take a look the `authProvider.ts` file </summary>
<p>


```ts title="src/authProvider.ts"
import { AuthProvider } from '@pankod/refine-core';
import { notification } from '@pankod/refine-antd';
import { supabaseClient } from 'utility';

const authProvider: AuthProvider = {
    login: async ({ email, password, providerName }) => {
        const { user, error } = await supabaseClient.auth.signIn({
            email,
            password,
            provider: providerName,
        });

        if (error) {
            return Promise.reject(error);
        }

        if (user) {
            return Promise.resolve();
        }

        // for third-party login
        return Promise.resolve(false);
    },
    register: async ({ email, password }) => {
        const { user, error } = await supabaseClient.auth.signUp({
            email,
            password,
        });

        if (error) {
            return Promise.reject(error);
        }

        if (user) {
            return Promise.resolve();
        }
    },
    forgotPassword: async ({ email }) => {
        const { data, error } =
            await supabaseClient.auth.api.resetPasswordForEmail(email, {
                redirectTo: `${window.location.origin}/update-password`,
            });

        if (error) {
            return Promise.reject(error);
        }

        if (data) {
            notification.open({
                type: 'success',
                message: 'Success',
                description:
                    "Please check your email for a link to reset your password. If it doesn't appear within a few minutes, check your spam folder.",
            });
            return Promise.resolve();
        }
    },
    updatePassword: async ({ password }) => {
        const { data, error } = await supabaseClient.auth.update({ password });

        if (error) {
            return Promise.reject(error);
        }

        if (data) {
            return Promise.resolve('/');
        }
    },
    logout: async () => {
        const { error } = await supabaseClient.auth.signOut();

        if (error) {
            return Promise.reject(error);
        }

        return Promise.resolve('/');
    },
    checkError: () => Promise.resolve(),
    checkAuth: async () => {
        const session = supabaseClient.auth.session();
        const sessionFromURL = await supabaseClient.auth.getSessionFromUrl();

        if (session || sessionFromURL?.data?.user) {
            return Promise.resolve();
        }

        return Promise.reject();
    },
    getPermissions: async () => {
        const user = supabaseClient.auth.user();

        if (user) {
            return Promise.resolve(user.role);
        }
    },
    getUserIdentity: async () => {
        const user = supabaseClient.auth.user();

        if (user) {
            return Promise.resolve({
                ...user,
                name: user.email,
            });
        }
    },
};

export default authProvider;
```

 </p>
</details>



:::tip 
Auth provider functions are also consumed by [refine authorization hooks](https://refine.dev/docs/api-reference/core/hooks/auth/useLogin/) under the hood. Since this is out of scope of this tutorial, we'll not cover them. 
:::


To activate auth provider, we need to register it to `authProvider` property of `<Refine>` component. 
We'll be using this provider to implement authentication and authorization features to our app. 




```tsx title="App.tsx"
import { Refine } from "@pankod/refine-core";
...
 // highlight-start
import authProvider from './authProvider';
// highlight-end

function App() {
  return (
    <Refine
      ...
      // highlight-next-line
      authProvider={authProvider}
      ...
    />
  );
}

export default App;
```


Also, we'll see `authprovider` methods in action when using `LoginPage` in the next sections.

 
At this point, our refine app is configured to communicate with Supabase API and perform authentication operations using Supabase Auth methods.

If you head over to localhost:3000, you'll see a welcome page.

<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={welcome} alt="welcome" />
</div>

<br/>

Now it's time to add some resources to our app.


## Adding CRUD pages   
Before we dive into Supabase features, we'll add simple CRUD pages to make out app interactive.

### Adding List post page

Let's add a list page to show data retrived from Supabase API. Copy and paste following code to `src/pages` folder and name it `posts.tsx`.


<details><summary>Show the code</summary>
<p>

```tsx title="src/pages/posts/index.tsx"
import { IResourceComponentsProps } from "@pankod/refine-core";

import {
    List,
    Table,
    useTable,
    Space,
    EditButton,
    ShowButton,
    getDefaultSortOrder,
    FilterDropdown,
    Select,
    useSelect,
} from "@pankod/refine-antd";

import { IPost, ICategory } from "interfaces";

export const PostList: React.FC<IResourceComponentsProps> = () => {
    const { tableProps, sorter } = useTable<IPost>({
        initialSorter: [
            {
                field: "id",
                order: "asc",
            },
        ],
        metaData: {
            select: "*, categories(title)",
        },
    });

    const { selectProps } = useSelect<ICategory>({
        resource: "categories",
    });

    return (
        <List>
            <Table {...tableProps} rowKey="id">
                <Table.Column
                    key="id"
                    dataIndex="id"
                    title="ID"
                    sorter
                    defaultSortOrder={getDefaultSortOrder("id", sorter)}
                />
                <Table.Column
                    key="title"
                    dataIndex="title"
                    title="Title"
                    sorter
                />
                <Table.Column
                    key="categoryId"
                    dataIndex={["categories", "title"]}
                    title="Category"
                    defaultSortOrder={getDefaultSortOrder(
                        "categories.title",
                        sorter,
                    )}
                    filterDropdown={(props) => (
                        <FilterDropdown {...props}>
                            <Select
                                style={{ minWidth: 200 }}
                                mode="multiple"
                                placeholder="Select Category"
                                {...selectProps}
                            />
                        </FilterDropdown>
                    )}
                />
                <Table.Column<IPost>
                    title="Actions"
                    dataIndex="actions"
                    render={(_, record) => (
                        <Space>
                            <EditButton
                                hideText
                                size="small"
                                recordItemId={record.id}
                            />
                            <ShowButton
                                hideText
                                size="small"
                                recordItemId={record.id}
                            />
                        </Space>
                    )}
                />
            </Table>
        </List>
    );
};
```

 </p>
</details>


:::info
Since this post focusing on Supase features, we'll not go into details of how to create a list page and how it works. You can refer to this [Tutorial section](https://refine.dev/docs/tutorials/ant-design-tutorial/#creating-a-list-page) to learn more about how to create a list page.
:::

### Adding Create post page

We'll add a create page to create new posts. Copy and paste following code to `src/pages` folder and name it `posts/create.tsx`.

<details><summary>Show the code</summary>
<p>

```tsx title="src/pages/posts/create.tsx"
import { useState } from "react";
import { IResourceComponentsProps } from "@pankod/refine-core";

import {
    Create,
    Form,
    Input,
    Select,
    Upload,
    useForm,
    useSelect,
    RcFile,
} from "@pankod/refine-antd";

import ReactMarkdown from "react-markdown";
import ReactMde from "react-mde";

import "react-mde/lib/styles/css/react-mde-all.css";

import { IPost, ICategory } from "interfaces";
import { supabaseClient, normalizeFile } from "utility";

export const PostCreate: React.FC<IResourceComponentsProps> = () => {
    const { formProps, saveButtonProps } = useForm<IPost>();

    const { selectProps: categorySelectProps } = useSelect<ICategory>({
        resource: "categories",
    });

    const [selectedTab, setSelectedTab] = useState<"write" | "preview">(
        "write",
    );

    return (
        <Create saveButtonProps={saveButtonProps}>
            <Form {...formProps} layout="vertical">
                <Form.Item
                    label="Title"
                    name="title"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Input />
                </Form.Item>
                <Form.Item
                    label="Category"
                    name="categoryId"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <Select {...categorySelectProps} />
                </Form.Item>
                <Form.Item
                    label="Content"
                    name="content"
                    rules={[
                        {
                            required: true,
                        },
                    ]}
                >
                    <ReactMde
                        selectedTab={selectedTab}
                        onTabChange={setSelectedTab}
                        generateMarkdownPreview={(markdown) =>
                            Promise.resolve(
                                <ReactMarkdown>{markdown}</ReactMarkdown>,
                            )
                        }
                    />
                </Form.Item>
                <Form.Item label="Images">
                    <Form.Item
                        name="images"
                        valuePropName="fileList"
                        normalize={normalizeFile}
                        noStyle
                    >
                        <Upload.Dragger
                            name="file"
                            listType="picture"
                            multiple
                            customRequest={async ({
                                file,
                                onError,
                                onSuccess,
                            }) => {
                                try {
                                    const rcFile = file as RcFile;
                                    await supabaseClient.storage
                                        .from("refine")
                                        .upload(`public/${rcFile.name}`, file, {
                                            cacheControl: "3600",
                                            upsert: true,
                                        });

                                    const { data } =
                                        await supabaseClient.storage
                                            .from("refine")
                                            .getPublicUrl(
                                                `public/${rcFile.name}`,
                                            );

                                    const xhr = new XMLHttpRequest();
                                    onSuccess &&
                                        onSuccess(
                                            { url: data?.publicURL },
                                            xhr,
                                        );
                                } catch (error) {
                                    onError &&
                                        onError(new Error("Upload Error"));
                                }
                            }}
                        >
                            <p className="ant-upload-text">
                                Drag & drop a file in this area
                            </p>
                        </Upload.Dragger>
                    </Form.Item>
                </Form.Item>
            </Form>
        </Create>
    );
};
```

 </p>
</details>

### Adding Interfaces and Normalize file

We need to add interfaces for `Post` and `Create` pages to `src/interfaces/index.d.ts` file.

<details><summary>Show the code</summary>
<p>

```tsx title="src/interfaces/index.d.ts"
export interface ICategory {
    id: string;
    title: string;
}

export interface IFile {
    name: string;
    percent: number;
    size: number;
    status: 'error' | 'success' | 'done' | 'uploading' | 'removed';
    type: string;
    uid: string;
    url: string;
}

export interface IPost {
    id: string;
    title: string;
    content: string;
    categoryId: string;
    images: IFile[];
}
```
 </p>
</details>

Also add `normalizeFile` function to `src/utility/normalize.ts` file.

<details><summary>Show the code</summary>
<p>

```tsx title="src/utility/normalize.ts"
import { UploadFile } from "@pankod/refine-antd";

interface UploadResponse {
    url: string;
}
interface EventArgs<T = UploadResponse> {
    file: UploadFile<T>;
    fileList: Array<UploadFile<T>>;
}

export const normalizeFile = (event: EventArgs) => {
    const { fileList } = event;

    return fileList.map((item) => {
        const { uid, name, type, size, response, percent, status } = item;

        return {
            uid,
            name,
            url: item.url || response?.url,
            type,
            size,
            percent,
            status,
        };
    });
};
```

</p>
</details>

### Adding Resources
One last thing we need to do is to add newly created pages to the `resources` prop of `Refine` component. 

```tsx title="src/App.tsx"
import { dataProvider } from '@pankod/refine-supabase';
import { supabaseClient } from 'utility';
//highlight-next-line
import { PostList, PostCreate } from 'pages/posts';

function App() {
    return (
        <Refine
            ...
            dataProvider={dataProvider(supabaseClient)}
            //highlight-start
            resources={[
                {
                    name: 'posts',
                    list: PostList,
                    create: PostCreate,
                },
            ]}
            //highlight-end
        />
    );
}

export default App;
```



- The `name` property refers to the name of the table in the Supabase database.

- The `list` property registers `/posts` endpoint to the `PostList` component.

- The `create` property registers `/posts/create` endpoint to the `PostCreate` component. Thereby, when you head over to `yourdomain.com/posts/create`, we should see the `PostCreate` page we just assigned.


[Refer to resources docs for more information &#8594](https://refine.dev/docs/api-reference/core/components/refine-config/#resources)


At this point our app will look like:



<div class="img-container">
    <div class="window">
        <div class="control red"></div>
        <div class="control orange"></div>
        <div class="control green"></div>
    </div>
    <img src={login} alt="login" />
</div>

<br/>


This is refine's default login screen is activated by giving `Loginpage` property to `Refine` component.

This is also where `authProvider` comes into play. At the [Auth Provider](#understanding-auth-provider) section, we added methods like "login",  "register" and "forgotPassword" methods to `authProvider`. 

These methods are linked to `LoginPage` component to perform authentication operations.




Sign in the app with followings credentials:


- email: info@refine.dev
- password: refine-supabase


