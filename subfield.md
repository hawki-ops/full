## Bug
In update field change the valiadtion schema for the fieldType
Select field error indicator
Fix the field update in the service "sameType"

## Dynamic Form

### util
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


// interface FormMetadata {
//     [fieldPath: string]: {
//         fieldID: number;
//         fieldType: string;
//         compositeFields : Composite[]
//     };
// }
            if ( field.type === "SUB" ){
                result[sectionName][fieldName] = {}
                field.compositeFields.forEach( (sub) => {
                    result[sectionName][fieldName][sub.name] = sub.type === "OPTIONS" ? 0 : '' 
                } )

            } else if (field.type === "OPTIONS") {
                result[sectionName][fieldName] = 0 ; // default to the first option
            } else {
                result[sectionName][fieldName] = '';
            }

            // Add field metadata to '$'
            const fieldPath = `${sectionName}.${fieldName}`;
            metadata[fieldPath] = {
                fieldID: field.id,
                fieldType: field.type,
                compositeFields: field.compositeFields || []
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

            if ( fieldType == "COMPOSITE" ){
                // Handle composite fields
                const compositeFields = metadata[fieldPath].compositeFields;
                const compositeValues = [];

                for (const subFieldName in fieldValue) {
                    const subField = compositeFields.find((cf) => cf.name === subFieldName);
                    if (subField) {
                        compositeValues.push({
                            compositeID: subField.id,
                            fieldType: subField.type,
                            value: fieldValue[subFieldName],
                        });
                    }
                }

                request.push({
                    fieldID,
                    fieldType: "SUB", // Identifier for composite fields
                    values: compositeValues,
                });

            }else{
                request.push({
                    fieldID,
                    fieldType,
                    value: fieldValue,
                });
            }

            
        }
    }

    return request;

}

```

### dynamic form 
```ts
"use client";
import { useFetch } from "@/app/context/FetchProvider";
import InputField from "@/app/shared/input-field";
import SelectField from "@/app/shared/select-field";
import { generateFormObject, prepareRequest } from "@/app/shared/utils";

import { Form, Formik } from "formik";
import React, { useEffect } from "react";

interface DynamicFormProps {
  sections: Section[];
}

const DynamicForm = (props: DynamicFormProps) => {
  const userID = 1;
  const { result, metadata } = generateFormObject(props.sections);
  const { post } = useFetch();

  if (Object.keys(result).length === 0) return null;

  const submitValues = async (values: any) => {
    const retrive = await post("api/values", {
      userID,
      values: prepareRequest(values, metadata),
    });
    if (retrive) {
      // successful
    }
  };

  return (
    <Formik
      initialValues={result}
      onSubmit={async (values, { setSubmitting }) => {
        await submitValues(values);
        setSubmitting(false);
      }}
    >
      {({ isSubmitting }) => (
        <Form>
          {props.sections &&
            props.sections.map((s, idx) => {
              return (
                <div
                  key={`s#${idx}`}
                  style={{ border: "1px solid black", margin: "10px" }}
                >
                  <h1> {s.name} </h1>
                  {s.fields.map((f, i) => {
                    if (f.type === "OPTIONS") {
                      return (
                        <SelectField
                          key={`f#${i}`}
                          label={f.name}
                          name={`${s.name}.${f.name}`}
                          options={f.options}
                        />
                      );
                    } else if (f.type === "COMPOSITE") {
                      return (
                        <div key={`f#${i}`} style={{ marginLeft: "10px" }}>
                          <h3>{f.name}</h3>
                          {f.compositeFields.map((cf, ci) => {
                            return cf.type === "OPTIONS" ? (
                              <SelectField
                                key={`cf#${ci}`}
                                label={cf.name}
                                name={`${s.name}.${f.name}.${cf.name}`}
                                options={cf.options || []}
                              />
                            ) : (
                              <InputField
                                key={`cf#${ci}`}
                                label={cf.name}
                                name={`${s.name}.${f.name}.${cf.name}`}
                                type={cf.type === "DATE" ? "date" : "text"}
                              />
                            );
                          })}
                        </div>
                      );
                    } else {
                      return (
                        <InputField
                          key={`f#${i}`}
                          label={f.name}
                          name={`${s.name}.${f.name}`}
                          type={f.type === "DATE" ? "date" : "text"}
                        />
                      );
                    }
                  })}
                </div>
              );
            })}

          <button type="submit" disabled={isSubmitting}>
            Submit
          </button>
        </Form>
      )}
    </Formik>
  );
};

export default DynamicForm;
```

### profile
```ts
"use client";
import React, { useEffect, useState } from "react";
import { useFetch } from "../context/FetchProvider";

const Profile = () => {
  const data = {
    "username": "JohnDoe",
    
      "sections": [
          {
              "name": "Personal Information",
              "fields": [
                {
                  "name" : "First Name",
                  "type" : "STRING",
                  "value" : "John"
                },
                  {
                      "name": "Adress",
                      "type": "SUB",
                      "value": null,
                      "values": [
                          {
                              "name": "Apartment",
                              "value": 15
                          },
                          {
                              "name": "Building",
                              "value": 30
                          }
                      ]
                  }
              ]
          }
      ]
  
  }
  // const [data, setData] = useState<any>({});
  // const userID = 1;

  // const { get } = useFetch();

  // const fetchInfo = async () => {
  //   const retrieved = await get(`api/values/user/${userID}`);
  //   setData(retrieved);
  // };

  // useEffect(() => {
  //   fetchInfo();
  // }, []);

  return (
    <div>
      {data && (
        <>
          <h1>Username: {data.username}</h1>
          {data.sections &&
            data.sections.length > 0 &&
            data.sections.map((s: any, idx: number) => {
              return (
                <div key={`ps#${idx}`}>
                  <h2>{s.name}</h2>
                  {s.fields.length > 0 && (
                    <ul>
                      {s.fields.map((f: any, i: number) => {
                        if (f.type === "SUB" && f.values && f.values.length > 0) {
                          // Handle "SUB" type fields
                          return (
                            <li key={`psf#${i}`}>
                              {f.name}:
                              <ul>
                                {f.values.map((subField: any, subIdx: number) => (
                                  <li key={`sub#${subIdx}`}>
                                    {subField.name}: {subField.value}
                                  </li>
                                ))}
                              </ul>
                            </li>
                          );
                        } else {
                          // Handle other field types
                          return (
                            <li key={`psf#${i}`}>
                              {f.name}: {f.value}
                            </li>
                          );
                        }
                      })}
                    </ul>
                  )}
                </div>
              );
            })}
        </>
      )}
    </div>
  );
};

export default Profile;
```



