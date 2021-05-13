---
title: "Submit static forms to Google Sheet: The Easy Way"
date: 2020-05-03T23:38:14+02:00
layout: post
---

When building static sites, you may not want to implement a backend for handling your form submissions.
Google sheet are a great way to store/use that data.

Here, I'm going to show you how to send your data, directy into a google sheet via a google form.

## Write a static form
Let's build a simple form, that we will use through this article.
Go ahead, and create a file (`index.html`), containing this code.
```
  <form>
    <div>
      <label for="first_name"> First Name </label>
      <input type="text" name="first_name"/>
    </div>

    <div>
      <label for="last_name"> Last Name </label>
      <input type="text" name="last_name"/>
    </div>
  </form>
```
## Setup a google form
If you've used google forms before, you know that the gathered responses can be exported to a google sheet.
We will be using a google form, as a middle man between our static form, and our google sheet. Think of it as a
messenger pigeon taking a message from Julius Ceasar (our static form), to his top General (our google sheet).

### Create google form
Head to [https://docs.google.com/forms](https://docs.google.com/forms), and create a google form with the same fields as our static form.

![Google Form](/public/img/google_form.png)

### Set a google sheet as response destination
Let's set the google form, so that responses go straight into a google sheet.
- Click on the "Responses" tab
- In the top right, click on the three-dot dropdown menu, and then select response destination
- Choose an option:
  - Create a new spreadsheet: Creates a spreadsheet for responses in Google Sheets
  - Select existing spreadsheet: Choose from your existing spreadsheets in Google Sheets to store responses

### Get prefilled link
This is a very interresting part.
- Click on the three-dot dropdown menu at the upper right of the Form editor, and choose "Get prefilled link"
- For each input, fill in the name attribute of the corresponding input in our static form
- Click  "Get Link", and copy that link somewhere

![Prefilled Google Form](/public/img/prefilled_google_form.png)


## Submit your google form through your static form
When someone submits our static form, we need to internally submit the google form with the same data, so that the informations can be saved in our google sheet.

We can submit a google form in one click, by calling its direct submit URL
A Google form direct submit URL is in the form: `https://docs.google.com/forms/d/e/FORMID/formResponse?&[entry.number=value]&submit=SUBMIT`.
We're going to call this url when our static form is submitted, by replacing the default values with our static form data.

You can see that the submit url is very similar to the prefill one. Let's tweak our prefill URL to get a submit URL.

In the prefill URL, replace `viewform?usp=pp_url` by `formResponse?` and then append `submit=SUBMIT` at the end of the url.

Now, let's supply our form data. If a user enters first_name = 'John', last name = 'Doe', our form action url will be:
`https://docs.google.com/forms/d/e/FORMID/formResponse?&entry.number=John&entry.number=Doe&submit=SUBMIT`.

Below is a full working example:
{% gist d9040faf287a7affdff064f8c83f5f4c %}

In this example, we use javascript `fetch` API, to submit the google form with the same data in our static form.

Now you can head to your google form or google sheet to see the responses.

Happy Hacking

#### References
- https://gist.github.com/unforswearing/7b01591138c7d3ca27f1b50182573233