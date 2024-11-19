## types
```ts
interface Option {
    id: number;
    name: string;
}

interface Field {
    id: number;
    name: string;
    fieldType: string;
    options: Option[];
}

interface Section {
    id: number;
    name: string;
    fields: Field[];
}

interface DynamicFormObject {
    [sectionName: string]: {
        [fieldName: string]: any;
    };
}

interface FormMetadata {
    [fieldPath: string]: {
        fieldID: number;
        fieldType: string;
    };
}
```

## Dynamic Form
```ts
"use client"
import React, { useEffect } from 'react';
import { Formik, Field, Form, ErrorMessage } from 'formik';
import { generateFormObject, prepareRequest } from './converter';
import Button from './button';
import { fetchForm } from './fetch';

const DynamicForm = ( ) => {
  const sections : Section[] = fetchForm();
  const { result, metadata } = generateFormObject(sections);

  useEffect(() => {
    console.log('Initial Form Object:', result);
    console.log('Form Metadata:', metadata);
  }, []);

  return (
    <Formik
      initialValues={result}
      onSubmit={(values) => {
        console.log('Form Submitted:', prepareRequest(values, metadata));
      }}
    >
      {({ setFieldValue }) => (
        <Form>
          {sections.map((section) => (
            <div key={section.id}>
              <h3>{section.name}</h3>
              {section.fields.map((field : any ) => {
                const fieldPath = `${section.name}.${field.name}`;
                return (
                  <div key={field.id}>
                    <label htmlFor={fieldPath}>{field.name}</label>
                    {field.fieldType === 'STRING' ? (
                      <Field
                        type="text"
                        id={fieldPath}
                        name={fieldPath}
                        placeholder={`Enter ${field.name}`}
                      />
                    ) : field.fieldType === 'OPTIONS' ? (
                      <Field as="select" name={fieldPath}>
                        {field.options.map((option : any ) => (
                          <option key={option.id} value={option.id}>
                            {option.name}
                          </option>
                        ))}
                      </Field>
                    ) : null}

                    <ErrorMessage name={fieldPath} component="div" />
                  </div>
                );
              })}
            </div>
          ))}
          <Button type="submit">Submit</Button>
        </Form>
      )}
    </Formik>
  );
};

export default DynamicForm;

```

## util
```ts
export function generateFormObject(obj: Section[]): { result: DynamicFormObject; metadata: FormMetadata } {
    const result: DynamicFormObject = {};
    const metadata: FormMetadata = {};

    obj.forEach((section) => {
        const sectionName = section.name;
        result[sectionName] = {};

        section.fields.forEach((field) => {
            const fieldName = field.name;

            // Set initial values based on field type
            if (field.fieldType === "OPTIONS") {
                result[sectionName][fieldName] = field.options[0].id ; // default to the first option
            } else {
                result[sectionName][fieldName] = '';
            }

            // Add field metadata to '$'
            const fieldPath = `${sectionName}.${fieldName}`;
            metadata[fieldPath] = {
                fieldID: field.id,
                fieldType: field.fieldType,
            };
        });
    });

    return { result, metadata };
}

export function prepareRequest( values : DynamicFormObject, metadata : FormMetadata) : any {
    const request = [];

    for (const sectionName in values) {
        const section = values[sectionName];
        for (const fieldName in section) {
            const fieldPath = `${sectionName}.${fieldName}`;
            const fieldID = metadata[fieldPath].fieldID;
            const fieldType = metadata[fieldPath].fieldType;
            const fieldValue = section[fieldName];

            request.push({
                fieldID,
                fieldType,
                value: fieldValue,
            });
        }
    }

    return request;

}

```
