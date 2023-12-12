# Cloud - Incomplete

commands
```bash
nc -vn 10.129.189.164 3690:
(UNKNOWN) [10.129.189.164] 3690 (svn) open
( success ( 2 2 ( ) ( edit-pipeline svndiff1 accepts-svndiff2 absent-entries commit-revprops depth log-revprops atomic-revprops partial-replay inherited-props ephemeral-txnprops file-revs-reverse list ) ) )
```
	
```bash
svn checkout svn://10.129.189.164
A    store
A    store/README.md
A    store/dynamo.py
A    store/sns.py
Checked out revision 5.
```

```bash
$ svn log svn://10.129.189.164
------------------------------------------------------------------------
r5 | root | 2022-06-14 05:59:42 -0400 (Tue, 14 Jun 2022) | 1 line

Adding database
------------------------------------------------------------------------
r4 | root | 2022-06-14 05:59:23 -0400 (Tue, 14 Jun 2022) | 1 line

Updating Notifications
------------------------------------------------------------------------
r3 | root | 2022-06-14 05:59:12 -0400 (Tue, 14 Jun 2022) | 1 line

Updating Notifications
------------------------------------------------------------------------
r2 | root | 2022-06-14 05:58:26 -0400 (Tue, 14 Jun 2022) | 1 line

Adding Notifications
------------------------------------------------------------------------
r1 | root | 2022-06-14 05:49:17 -0400 (Tue, 14 Jun 2022) | 1 line

Initializing repo
------------------------------------------------------------------------
```

```bash
import boto3

client = boto3.client('dynamodb',
               region_name='us-east-2',
               endpoint_url='http://cloud.htb',
               aws_access_key_id='',
               aws_secret_access_key=''
               )

client.create_table(TableName='users',
        KeySchema=[
            {
                'AttributeName': 'username',
                'KeyType': 'HASH'
            },
            {
                'AttributeName': 'password',
                'KeyType': 'RANGE'
            },
        ],
        AttributeDefinitions=[
            {
                'AttributeName': 'username',
                'AttributeType': 'S'
            },
            {
                'AttributeName': 'password',
                'AttributeType': 'S'
            },
        ],
        ProvisionedThroughput={
            'ReadCapacityUnits': 5,
            'WriteCapacityUnits': 5,
        }
        )


client.put_item(TableName='users',
        Item={
                'username': {
                        'S': 'marcus'
                },
                'password': {
                        'S': 'dFc42BvUs02'
                },
        }
        )

```

```bash
```

```bash
```