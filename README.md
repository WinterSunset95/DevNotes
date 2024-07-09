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
