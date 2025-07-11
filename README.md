# Safe Shield
Safe Shield is a robust Python library designed to validating HTTP requests and input data.

This project is licensed under the AGPLv3 License - see the [LICENSE](LICENSE) file for details

## Table of Contents
- [Installation](#installation)
- [Overview](#usage)
  - [Usage](#usage)
  - [Validation Data](#validation-data)
  - [Error Messages](#error-messages)
  - [Attributes Custom](#attributes-custom)
  - [Rules](#rules)
  - [Custom Rule](#custom-rule)
- [Example](#example)
- [LICENSE](LICENSE)

## Installation
Use the package manager [pip](https://pypi.org/project/safeshield/) to isntall Safe Shield.
```bash
pip install safeshield
```

## Usage
User should pass request dictionary and rules dictionary for validation data in the request.

Please see examples below:

```bash
from validator import Validator

request = {
  "name": "Wunsun",
  "email": "wunsun58@gmail.com",
  "age": 20,
}

rules = {
  "name": "required",
  "email": "required|email",
  "age": "required|integer|min:18",
}

validated_data = Validator(request, rules).validate() # Validated data returned
```

Validator().validate() returns either validated data or False

## Validation Data
### Rule Format
You can define multiple rules for each field as a string, tuple, or list.
- String Format
   
```bash
rules = {
  "name": "required",
  "email": "required|email",
  "age": "required|integer|min:18",
}
```

- List Format

```bash
rules = {
  "name": ["required"],
  "email": ["required"， "email"],
  "age": ["required"，"integer", "min:18"],
}
```

- Tuple Format

```bash
rules = {
  "name": ("required"),
  "email": ("required"， "email"),
  "age": ("required"，"integer", "min:18"),
}
```  

All rules can be invoked either as a class instantiation or via method calls from the Rule class (e.g., Required()) or methods (e.g., Rule.required()).

```bash
from validator.rules import Rule, Required, Email

rules = {
  "name": Required(),  # Single rule (no brackets needed)
  "email": [Required(), Email()],  # Initialization Rule Class
  "age": [Rule.required()，Rule.integer(), Rule.min(18)],  # Call rule from Rule Class
}
```

### Nested Rules:
Nested data validation, use dot notation like 'user.name' to access deep fields or wildcards like 'orders.*.id' to validate all array items. The system automatically checks all nested levels and returns clear error messages pointing to exact validation failures."

```bash
rules = {
    "user.name": Required(),  # Top-level field
    "user.contact.email": [Required(), Email()],  # Nested field
    "orders.*.id": Required(),  # Wildcard for list items
}
```

## Error Messages
This validator enables users to track validation failures and receive corresponding error messages.

### Basic Error Validation
This demonstrates fundamental validation failures for required fields and data types:
```bash
from validator import Validator

request = {
  "name": "",  # Empty value
  "email": "wunsun58@gmail.com",
]

rules = {
  "name": "required|string"
}

validator = Validator(request, rules)
validator.validate()

"""
validator.has_errors -> True
validator.errors:
  {
    "name": ["The name field is required.", "The name must be a string"]
  }
"""
```

- Output Explanation:
  - validator.has_errors → True (Indicates validation failed)
  - validator.errors → Returns detailed error messages:
    - For the name field:
      - "The name field is required." - Because the value is empty
      - "The name must be a string." - Empty string fails type validation

### Nested Error Validation
Validates complex nested structures (objects/arrays) with precise error location:
```bash
from validator import Validator

data = {
    "user": {
        "name": "Alice",
        "contact": {"email": "alice@example.com"} # Invalid dns email format
    },
    "orders": [
        {"id": 1, "price": "One hundred"}, # Invalid price type
        {"id": 2, "price": 200}  # Valid entry
    ]
}

rules = {
  "user.name": "required",
  "user.contact.email": "email:dns",  # Requires valid DNS email records
  "orders.*.price": "required|integer", # Wildcard validates all array item
}

validator = Validator(request, rules)
validator.validate()

"""
validator.has_errors -> True
validator.errors:
  {
    "user.contact.email": ["The email must be a valid email with valid DNS records."],
    "orders.0.price": ["The price must be a number"]
  }
"""
```

- Output Explanation:
  - validator.has_errors → True
  - validator.errors → Pinpoints exact failures:
    - user.contact.email: Fails DNS validation
    - orders.0.price: First array item fails integer validation
    - Note how array indices (0, 1, etc.) are automatically identified

### Custom Error Message
Override default messages with user-friendly alternatives:
```bash
messages = {
  "user.contact.email": "This email is invalid"
}

validator = Validator(request, rules, messages)
validator.validate()

"""
validator.has_errors -> True
validator.errors:
  {
    "user.contact.email": ["This email is invalid"],
    "orders.0.price": ["The price must be a number"]
  }
"""
```
