# Автоматизація розсилки електронних листів за подіями у DynamoDB

Цей README описує процес налаштування середовища AWS для автоматичної розсилки електронних листів реагуючи на події в DynamoDB.

## Крок 1: Підготовка DynamoDB

### Створення таблиці DynamoDB

1. Перейдіть у AWS Management Console і відкрийте службу DynamoDB.
2. Натисніть "Create table".
3. Введіть ім'я таблиці, наприклад `Users`.
4. Встановіть первинний ключ `userId` типу String.
5. Створіть пару атрибутів: email, name
6. Збережіть налаштування таблиці.

### Додавання записів до таблиці

Використовуйте AWS CLI або AWS Management Console для додавання записів з атрибутами `email`, `name`, та іншими необхідними полями.

## Крок 2: Налаштування DynamoDB Streams

1. У налаштуваннях вашої таблиці `Users` активуйте DynamoDB Streams.
2. Виберіть "New and old images" як тип даних потоку.

## Крок 3: Створення Lambda Функції

1. У AWS Management Console перейдіть до AWS Lambda і створіть нову функцію.
2. Виберіть Python як мову виконання.
3. Налаштуйте тригер, вибравши DynamoDB Streams як джерело подій.
4. Вкажіть потік, пов'язаний з вашою таблицею `Users`.
5. Надайте функції необхідні дозволи IAM через роль.

## Крок 4: Налаштування Amazon SES

1. Переконайтеся, що ви підтвердили вашу електронну адресу в Amazon SES. Якщо використовуєте адресу, треба підтвердити в identities адреси відправника і отримувачів. 
2. Створіть шаблон електронного листа за потреби.

## Крок 5: Написання коду Lambda Функції

```python
import boto3
import json

ses_client = boto3.client('ses')
sender_email = 'your-verified-email@example.com'

def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'INSERT':
            new_image = record['dynamodb']['NewImage']
            user_email = new_image['email']['S']
            user_name = new_image['name']['S']
            
            response = ses_client.send_email(
                Source=sender_email,
                Destination={'ToAddresses': [user_email]},
                Message={
                    'Subject': {'Data': 'Welcome!'},
                    'Body': {'Text': {'Data': f'Hello, {user_name}! Welcome to our service.'}}
                }
            )
            print(f"Email sent! Message ID: {response['MessageId']}")
```

Замініть your-verified-email@example.com на вашу підтверджену адресу в Amazon SES.

## Перевірка журналів виконання
Якщо ваша Lambda функція не працює як очікувалося, перевірте журнали в Amazon CloudWatch, щоб ідентифікувати можливі проблеми. Знайти можна в log groups > lambda/functionName.

## Заключні зауваження
Переконайтеся, що у вашій Lambda функції є відповідні дозволи (роль можливо створити коли створюєте базу даних, потім додати відповідні дозвони - читати стрім, писати листи)
