## Domo Data Features

Using the Toolbox a user can select the following Domo data components to use in their survey.

### Single Select and Multi Select Domo Dropdowns

- Choose **Domo Dropdown** for single select or **Multi-Select Domo Dropdown** for multi-select.
- Navigating to the **Property Editor**, find **Choices from a Domo Datasource**.
- Select a Domo datasource.
- Choose value to store from a property in the dataset.
- Choose value to display from a property in the dataset.

### File Upload

File upload question utilizes Domo's file storage.

### Dynamic Matrix Bulk Entry and Single Line Entry

- **Bulk Entry**: Group of answers are saved by placing them in a single column.
- **Single Line**: Each row and column pair are saved in their own columns.

### Domo Variables

Users will have preset variables available to them.

- Domo User ID.
- Domo User Email

Users can then use the following to create variables:

- Selecting **Create Domo variable expressions**.
- Choose a **Domo Datasource**.
- Choose a **Compare Column** to select a column from the dataset.
- Choose a **Form value** to select a variable from the form.
- (Optional) **Add Condition**, to filter data further.
- **Select value to store** from the datasource whose row will be used as the Domo Variable.
- **Select alias** for the Domo Variable.
- Click **Add** or hit Enter to add Domo Variable.
- Click **Apply**.

Enable Export for Domo Variable.

- Check the checkbox of a variable to have it exported to the form’s dataset with each user’s submission.

Set Question's value using Domo Datasource.

- Selecting **Set Value using Domo Datasources**.
- Choose **Compare Column** to select a column from the dataset.
- Choose **Form value** to select a Domo Variable from the form.
- (Optional) **Add Condition**, to filter data further.
- Choose column or Domo Variable to use as Prefill.
- Click **Apply**.

### Domo Groups

Using Domo Groups a user can set the visibility and readability of a panel, page or question. Further more a user can also set a value of a question.

Using the following:

- **Make the question visible to these groups**, groups that can see a question if the visibility is unchecked.
- **Disable the read only mode for groups**, groups that can edit a question if the readability is checked.
- **Set value for these groups** with **Set value expression**, setting the value for groups using expressions.
