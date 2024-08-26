Adding a New Form

This guide outlines the process for adding a new form to our app, along with best practices, folder structure, and important notes. Our forms are built using React, react-hook-form, Zod for validation, and Shadcn components for UI elements.

Folder Structure
To maintain consistency across the app, follow this folder structure when adding a new form:
```
src/
|
|__pages/
     |__[PageName]/
          |__[FormName]/
          |     |__[FormName]Component.tsx # Wrapper component for form, handles data fetching and other logic
          |     |__Form.tsx # Contains the input fields, form structure, useForm instance and onSubmit
          |__Models/
               |__Schema.ts
               |__index.ts
```

Steps to Add a New Form
1.Create the Zod Schema

- Define the validation schema for the form using Zod.
- Create a new file `[FormName].ts` inside of `./Models`.
- Use `zod` schema helper functions wherever necessary, they can be found in `./src/utils/form.ts`

```
// ./Models/Schema1.ts
import { z } from "zod";
import { Schema2 } from "./Schema2";
import { Schema3 } from "./Schema3";
import { positiveNumber, nonEmptyString, positiveInteger } from "@/utils/form";

export const ServiceSchema = z.object({
  name: nonEmptyString("Name"),
  hourlyWage: positiveNumber("Wage"),
  hoursBriefingPerEmployee: positiveInteger("Hours Briefing per Employee"),
  siteId: z.number().optional(),
  payGroup: z.enum(["Group 1", "Group 2", "Group 3"], {
    message: "Please select a pay group.",
  }),
  nonTariffAllowance: positiveNumber("Non-Tariff Allowance"),
  shifts: z.array(Schema2).min(1, { message: "At least one shift must be added" }),
  margin: positiveNumber("Margin"),
  pauseIncluded: z.boolean().default(true),
  siteInput: Schema3.optional(),
});

export const ServiceDefaultValues = {
  name: "",
  siteId: undefined,
  hourlyWage: undefined,
  // type: ServiceTypeEnum.Permanent,
  nonTariffAllowance: undefined,
  shifts: [],
  margin: 0,
  pauseIncluded: true,
  siteInput: undefined,
  payGroup: undefined,
};

export type TServiceSchema = z.infer<typeof ServiceSchema>;
```
2.Build the Form Component

- Create the main form component in Form.tsx.
- Use react-hook-form to manage form state.
- Register fields using register() with Shadcn components.
- If applicable, use TextInfoGroups for non-editable fields, similar to the EditMobileService form setup.

```
// src/pages/[PageName]/[FormName]/Form.tsx

import React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { ServiceSchema } from '../Models';
import { Service } from '../Models';

const Form: React.FC<FormProps> = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<TSchema>({
    resolver: zodResolver(Schema),
  });

  /**
   * OnSubmit handler
   * @param {TSchema} data
   * @returns
   */
  const handleFormSubmitt: SubmitHandler<TSchema> = (data: Schema): void => {
    // Handle form submission logic here
    console.log('Form Data:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Name:</label>
        <input {...register('name')} />
        {errors.name && <span>{errors.name.message}</span>}
      </div>
      <div>
        <label>Price:</label>
        <input type="number" {...register('price')} />
        {errors.price && <span>{errors.price.message}</span>}
      </div>
      {/* Add more fields as necessary */}
      <button type="submit">Submit</button>
    </form>
  );
};

export default Form;
```
3.Build the Form Wrapper Component

```
// src/pages/[PageName]/[FormName]/[FormName]Component.tsx

import React from 'react';
import Form from './Form';
import { ActionButtonConfig } from "@/components/shared/FormGroupHeader";

const FormNameComponent: React.FC = () => {

  // Queries
  
  // Mutations

  // Other necessary logic

  /**
   * OnClick handler for the Back button
  */
  const handleBackClick = () => navigate(-1);

  // Action buttons for FormGroupHeader
  const actionButtons: ActionButtonConfig[] = [
    {
      text: "Back",
      actionFn: handleBackClick,
      variant: "outline",
    },
    {
      text: "Submit",
      type: "submit",
      form: "add-service-form", // Set id of form inside of ./Form.tsx
    }
  ];

  return (
    <>
      FormGroupHeader
        breadcrumbs={<Breadcrumbs />} // If needed
        actionButtons={actionButtons} // If needed
        title={"Form Title"}
        description={"Some descriptive text"}
      />

      <Form {...props} /> // If any
    </>
  );
};

export default FormNameComponent;
```

### Best Practices
- **Schema-Driven Forms**: Use Zod schemas to drive form validation and type safety. This ensures that validation logic is consistent and centralized.
- **Modular Components**: Break down your form into smaller subcomponents for better maintainability and reusability.
- **Use of Shared Components**: Utilize shared components like FormGroupHeader, VisitAmountsDisplay, and DayInputPlaceHolder for consistent UI and behavior across forms.
- **State Management**: Keep form state localized within the form component or use context/hooks for managing complex state that needs to be shared across components.
- **Error Handling**: Implement comprehensive error handling using react-hook-form’s formState.errors and display error messages near the relevant fields.

### Notes
- **Shadcn Components**: Use Shadcn UI components for inputs, buttons, and dropdowns to maintain a consistent design language.
- **Non-Editable Fields**: For fields that should not be editable (like in EditMobileService), use TextInfoGroups to display the information.
- **Action Buttons**: The FormGroupHeader supports dropdown action buttons if needed. Refer to the `ActionButtonConfig` type for usage.

### Example Forms in the App
- **AddServiceForm**: Located in `./src/pages/EditTender/AddService/*`. This form make use of a nested form, `AddShift`
- **EditMobileServiceForm**: Located in components/forms/EditMobileServiceForm/. This form only allows editing specific fields (`carCostPerMinute` and `otherCostPerMinute`) and uses `TextInfoGroups` for the rest.