[![npm version](https://badge.fury.io/js/react-form-ctl.svg)](https://badge.fury.io/js/react-form-ctl)

# React Form Ctl

React Form Ctl is a **simple** and **type-safe** way to handle form values and validation. It is inspired by Angular's `FormControl` and/or `FormBuilder`.

## Basic Usage

```tsx
import {useFormCtl, Validators} from 'react-form-ctl';

type FormData = {
    name: string;
    age: number;
    isOldEnough: boolean;
}

const FormComponent = () => {
    const {data} = useFormCtl<FormData>({
        name: ['John', [Validators.required]],
        age: [21],
        isOldEnough: [true],
    });

    return <>
        <input
            type="text"
            checked={data.isOldEnough.name}
            onChange={e => {
                data.name.setValue(e.target.value);
            }}
        />

        {/* ... */}
    </>
}
```

## Error Handling

In case you want to write a more specific error message for different errors, it is recommended to supply a Map with your error messages like in the following example. Note that this will only be applied to the first error supplied by the first failing Validator.

```tsx
import {useFormCtl, extError, Validators, ErrorMappingsType} from 'react-form-ctl';

const {data} = useFormCtl<FormData>({
    name: ['John', [Validators.required, Validators.minLength(5)]],
    // ...
});

const errorMap: ErrorMappingsType = {
    required: () => 'Field is required',
    minLength: ({length, expectedLength}) => `Minimum Length: ${length}/${expectedLength}`
};

return <>
    {/* input field */}

    { !data.name.valid && <div>
        {extError(errorMap, data.name.error)}
    </div>}
</>
```

Alternatively, you can also get all errors manually and implement your own error handling like so:

```tsx
import {useFormCtl, Validators} from 'react-form-ctl';

const {data} = useFormCtl<FormData>({
    name: ['John', [Validators.required, Validators.minLength(5)]],
    // ...
});

return <>
    {/* input field */}

    { data.name.errors.required && <div>
        Field is required
    </div>}
    { data.name.errors.minLength && <div>
        Minimum Length: {data.name.errors.minLength.length}/{data.name.errors.minLength.expectedLength}
    </div>}
</>
```

## Validators

There are a number of Validators already included, which are the following:

- required
- minLength
- maxLength

You can also implement your own validators and pass them to the Validators array:

```tsx
import {useFormCtl, Validators, ValidatorType} from 'react-form-ctl';

// A simple custom validator
const nameNotBlacklisted: ValidatorType = (value: any) => {
    if(['Max', 'Anna'].includes(value)) {
        return {
            name: 'nameNotBlacklisted',
            found: value
        };
    }

    return null;
}

// You can also create parametrized custom validators
const isExactAge: (age: number): ValidatorType => {
    return (value: number) => {
        if (value !== age) {
            return {
                name: 'isExactAge',
                expected: age
            };
        }

        return null;
    };
},

// Then use them like other validators inside your Validators array
const {data} = useFormCtl<FormData>({
    name: ['John', [nameNotBlacklisted]],
    age: [21, [isExactAge(42)]],
    // ...
});

// Don't forget to also write custom error handlers if you want
const errorMap: ErrorMappingsType = {
    nameNotBlacklisted: ({found}) => `Name is blacklisted: ${found}`,
    isExactAge: ({expected}) => `Expected age of ${expected}`
};
```