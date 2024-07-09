# Notes taken on my developer journey
## WinterSunset95

- [Append to a list on AWS Dynamodb](https://stackoverflow.com/questions/31288085/how-to-append-a-value-to-list-attribute-on-aws-dynamodb)
```
import { DynamoDBClient, UpdateItemCommand } from '@aws-sdk/client-dynamodb'
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb'

const dbClient = new DynamoDBClient()
const docClient = DynamoDBDocumentClient.from(dbClient)

const command = new UpdateItemCommand({
    TableName: 'my-table',
    Key: { id: { S: 'my-id' } },
    UpdateExpression: 'SET myList = list_append(myList, :myValue)',
    ExpressionAttributeNames: { 
        'myList': { L: [{ S: 'value1' }] }
    },
})

await docClient.send(command)
```

- [Setup next-auth on app router with Cognito](https://next-auth.js.org/providers/aws-cognito)
/api/auth/[...nextauth]/route.ts
```
import NextAuth from 'next-auth'
import CognitoProvider from 'next-auth/providers/cognito'

const handler = NextAuth({
    providers: [
        CognitoProvider({
            clientId: process.env.COGNITO_CLIENT_ID,    // Get from aws
            clientSecret: process.env.COGNITO_CLIENT_SECRET,    // Get from aws
            issuer: process.env.COGNITO_ISSUER,    // Get from aws
        }),
    ]
})
```
Environment variables NextAuth uses:
- NEXTAUTH_URL - Base URL of the site (e.g. https://example.com for prod or http://localhost:3000 for dev)
- NEXTAUTH_SECRET - Not needed for development, but required for production
On client components, use `useSession` hook to get session data.
To invoke the login, use `signIn` method from `next-auth/client`.
example.tsx
```
import { signIn } from 'next-auth/client'

export default function SignIn() {
    return (
        <button onClick={() => signIn()}>Sign in</button>
    )
}
```

- [Use next-auth on a server page](#auth)
If you already have the /api/auth/[...nextauth]/route.ts file, you can use the `getServerSession` method from `next-auth` to get the session data.
/admin.tsx
```
import { getServerSession } from 'next-auth/server'

export async function Page() {
    const session = await getServerSession()
    if (!session) {
        return {
            redirect: {
                destination: '/',
                permanent: false,
            },
        }
    }
    return {
        <div>
            <h1>Admin page</h1>
        </div>
    }
}
```

- [Get an item from a DynamoDB table](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/programming-with-javascript.html#programming-with-javascript-high-level-client)
```
const command = new GetCommand({
    TableName: 'my-table',
    Key: { id: { S: 'my-id' } },
})
```

- Handle image uploads with Next.js
Recieve the file, and then convert it to a buffer/blob
```
const handleImageChange = (e) => {
    const file = e.target.files[0]
    const url = URL.createObjectURL(file)
}
```
You can then use the `url` to display the image on the client side.
