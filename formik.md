```ts
"use client"
import { Formik, Form, Field, FieldArray, ErrorMessage, FieldArrayRenderProps } from 'formik';
import * as Yup from 'yup';

// Define the shape of the form values
interface Item {
  label: string;
  value: string;
}

interface FormValues {
  name: string;
  items: Item[];
}

const MyForm: React.FC = () => {
  // Initial values
  const initialValues: FormValues = {
    name: '',
    items: [
      { label: 'Item 1', value: '' },
      { label: 'Item 2', value: '' },
    ],
  };

  // Validation schema
  const validationSchema = Yup.object({
    name: Yup.string().required('Name is required'),
    items: Yup.array().of(
      Yup.object({
        label: Yup.string().required('Label is required'),
        value: Yup.string().required('Value is required'),
      })
    ),
  });

  // Submit handler
  const handleSubmit = (values: FormValues) => {
    console.log('Form values:', values);
  };

  return (
    <Formik
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={handleSubmit}
    >
      {({ values }: { values: FormValues }) => (
        <Form>
          <div>
            <label htmlFor="name">Name</label>
            <Field id="name" name="name" placeholder="Enter your name" />
            <ErrorMessage name="name" component="div" className="error" />
          </div>

          <FieldArray name="items">
            {({ push, remove }: FieldArrayRenderProps) => (
              <div>
                {values.items.map((item, index) => (
                  <div key={index}>
                    <label htmlFor={`items.${index}.label`}>Label</label>
                    <Field
                      name={`items.${index}.label`}
                      placeholder="Label"
                    />
                    <ErrorMessage name={`items.${index}.label`} component="div" className="error" />

                    <label htmlFor={`items.${index}.value`}>Value</label>
                    <Field
                      name={`items.${index}.value`}
                      placeholder="Value"
                    />
                    <ErrorMessage name={`items.${index}.value`} component="div" className="error" />

                    <button type="button" onClick={() => remove(index)}>
                      Remove
                    </button>
                  </div>
                ))}
                <button
                  type="button"
                  onClick={() =>
                    push({ label: `Item ${values.items.length + 1}`, value: '' })
                  }
                >
                  Add Item
                </button>
              </div>
            )}
          </FieldArray>

          <button type="submit">Submit</button>
        </Form>
      )}
    </Formik>
  );
};

export default MyForm;

```
