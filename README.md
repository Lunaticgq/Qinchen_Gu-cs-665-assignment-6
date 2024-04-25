
| CS-665       | Software Design & Patterns |
|--------------|----------------------------|
| Name         | Qinchen Gu                 |
| Date         | 04/23/2024                 |
| Course       | Spring                     |
| Assignment # | 6                          |

(I choose Assignment 3 for this assignment)

# GitHub Repository Link:
https://github.com/Lunaticgq/Qinchen_Gu-cs-665-assignment-6

# Implementation Description
## How is Flexibility?
The design of the application is inherently flexible. It's easy to introduce new customer types or modify existing ones without having to overhaul the core logic. This is primarily due to the use of the strategy pattern and a clear separation of concerns, making the application adaptable to future requirements or changes.

## How are Simplicity and Understandability?
The application with separate classes for customers and email templates, ensures that each class has a single responsibility. And by adopting the strategy pattern, the different methods of generating emails for different customer types are clearly separated into individual customer classes. This clear delineation enhances the readability of the code, making it easier to understand. And adding a new customer type doesn't require deep changes in other parts of the system.

## How you avoid duplicated code?
The strategy pattern is applied to encapsulate varying behaviors based on customer types. Instead of having a monolithic method with conditional logic to handle each customer type, each behavior is encapsulated in its corresponding subclass. This prevents code duplication and makes the behavior extensible. By wrapping data and methods inside classes and using proper access modifiers, we ensure that code is not inadvertently duplicated elsewhere.

## Why avoid duplicated code is important?
Since less code there are, more readable it is, and it will be easier for debugging. Also, this action can make modify
of code much easier.

## Design Pattern
I used Strategy Pattern in this application. The strategy pattern is applied in the context of generating email content for different types of customers. It allows for the easy addition or removal of customer types in the future. Strategy pattern ensure changes in the requirements or the addition of new functionalities would have minimal impact on the existing code.

# Details about works I did for this Assignment
## Examine your code and identify opportunities for code improvement

### Problem 1: All customer share same `EmailTemplate` instance.

```jsx
package EmailGenerationApp;

public class EmailTemplate {
    private String template = "Dear [CUSTOMER_TYPE],\n\n[MESSAGE]\n\nRegards,\nThe Company";

    public String getTemplate() {
        return template;
    }
}

```

 This can be problematic if you want to customize the template for different types of customers.

### Problem 2: There are something hard-code in the source code.

If we want to change the template of some customer, we need to recompile the whole project. When the project is getting larger. It could be harmful.

1. Template is hard code in the source code.
    
    ```java
    // Hard code email template.
    public class EmailTemplate {
        private String template = "Dear [CUSTOMER_TYPE],\n\n[MESSAGE]\n\nRegards,\nThe Company";
    
        public String getTemplate() {
            return template;
        }
    }
    ```
    
2. Message is hard code in the source code.
    
    ```java
        // Hard code part in class BusinessCustomer.
        public String generateEmail() {
            String email = emailTemplate.getTemplate();
            return email.replace("[CUSTOMER_TYPE]", "Business Customer " + name)
                    .replace("[MESSAGE]", "Message for Business Customer");
        }
    ```
    

### Problem 3 : Not clear how many type of customer.

If we want to be clear how many type of our customer, we need to know it. If we want to have some operations target to a specific type of customer. We must pre-define those kind of customer. We need some more struct method.

## Solution for the problem:

In my new structure, creating a email as following step, use business class as an example:

- First, `BusinessCusomer` class will invoke `generateEmail` method in `EmailFactory` class, and pass `type = CustomerType.Bussiness` in it.
- `EmailFactory` will invoke `generateEmail` in `Email` class, which will do the real generating job.
- `Email` class will generate a `EmailTemplateConfig` object to read template, message and customer type to generate a whole email.
- `EmailTempateConfig` read configuration from `[emailTemplates.properties](http://emailTemplates.properties)` file.
- When the `email` return to `BusinessCustomer` class, it will replace NAME to the customer name.

In my new the main change in my code as following.(I also change other classes to fit this change.)

### 1. Add a factory class to generate the Email according to the customer type.

```java
public class EmailFactory {

    public static String getEmail(CustomerType type) {

        return new Email().generateEmail(type);
    }
}

```

### 2. Implement an configuration class, to get config from other source, in my code, from `[emailTemplates.properties](http://emailTemplates.properties)` .

```java
public class EmailTemplateConfig {
    private Properties templates;

    public EmailTemplateConfig() {
        templates = new Properties();
        loadTemplates();
    }

    private void loadTemplates() {
        try (InputStream input = getClass().getClassLoader().getResourceAsStream("emailTemplates.properties")) {
            if (input == null) {
                throw new IOException("Unable to find emailTemplates.properties");
            }
            templates.load(input);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public String getTemplate(String key) {
        return templates.getProperty(key);
    }
}
```

### 3. Add CustomerType class to manage the type of customer.

```java
package EmailGenerationApp;

public enum CustomerType {
    BUSINESS, VIP, RETURNING, FREQUENT, NEW;

    @Override
    public String toString() {
        switch (this) {
            case BUSINESS:
                return "Business";
            case VIP:
                return "VIP";
            case RETURNING:
                return "Returning";
            case FREQUENT:
                return "Frequent";
            case NEW:
                return "New";
            default:
                throw new IllegalArgumentException();
        }
    }
}

```
