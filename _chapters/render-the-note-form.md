---
layout: post
title: Render the Note Form
date: 2017-01-29 00:00:00
lang: en
description: We are going to render a user’s note in a form in our React.js app. To render the form fields, we’ll use React-Bootstrap’s FormGroup and FormControl components.
comments_id: render-the-note-form/140
ref: render-the-note-form
---

Now that our container loads a note using the 'useEffect()' method, let's go ahead and render the form that we'll use to edit it.

<img class="code-marker" src="/assets/s.png" />Replace our placeholder `return` statement in `src/containers/Notes.tsx` with the following.

```coffee
  const validateForm = () => {
    return content.length > 0;
  };

  const formatFilename = (str: string) => {
    return str.replace(/^\w+-/, "");
  };

  const handleFileChange = (
    event: React.FormEvent<FormControl> & CustomFormEvent
  ) => {
    file.current = event.target.files[0];
  };

  async function handleSubmit(
    event: React.FormEvent<HTMLFormElement & CustomFormEvent>
  ) {
    let attachment;

    event.preventDefault();

    if (file.current && file.current.size > config.MAX_ATTACHMENT_SIZE) {
      alert(
        `Please pick a file smaller than ${config.MAX_ATTACHMENT_SIZE /
          1000000} MB.`
      );
      return;
    }

    setIsLoading(true);
  }

  async function handleDelete(
    event: React.MouseEvent<Button, MouseEvent> & CustomFormEvent
  ) {
    event.preventDefault();

    const confirmed = window.confirm(
      "Are you sure you want to delete this note?"
    );

    if (!confirmed) {
      return;
    }

    setIsDeleting(true);
  }

  return (
    <div className="Notes">
      {note && (
        <form onSubmit={handleSubmit}>
          <FormGroup controlId="content">
            <FormControl
              value={content}
              componentClass="textarea"
              onChange={(e: React.FormEvent<FormControl> & CustomFormEvent) =>
                setContent(e.target.value)
              }
            />
          </FormGroup>
          {note.attachment && (
            <FormGroup>
              <ControlLabel>Attachment</ControlLabel>
              <FormControl.Static>
                <a
                  target="_blank"
                  rel="noopener noreferrer"
                  href={note.attachmentURL}
                >
                  {formatFilename(note.attachment)}
                </a>
              </FormControl.Static>
            </FormGroup>
          )}
          <FormGroup controlId="file">
            {!note.attachment && <ControlLabel>Attachment</ControlLabel>}
            <FormControl onChange={handleFileChange} type="file" />
          </FormGroup>
          <LoaderButton
            block
            type="submit"
            bsSize="large"
            bsStyle="primary"
            isLoading={isLoading}
            disabled={!validateForm()}
          >
            Save
          </LoaderButton>
          <LoaderButton
            block
            bsSize="large"
            bsStyle="danger"
            onClick={handleDelete}
            isLoading={isDeleting}
          >
            Delete
          </LoaderButton>
        </form>
      )}
    </div>
  );
```

We are doing a few things here:

1. We render our form only when the `note` state variable is set.

2. Inside the form we conditionally render the part where we display the attachment by using `note.attachment`.

3. We format the attachment URL using `formatFilename` by stripping the timestamp we had added to the filename while uploading it.

4. We also added a delete button to allow users to delete the note. And just like the submit button it too needs a flag that signals that the call is in progress. We call it `isDeleting`.

5. We handle attachments with a file input exactly like we did in the `NewNote` component.

6. Our delete button also confirms with the user if they want to delete the note using the browser's `confirm` dialog.

Further Add some CustomeTypes by creating `libs/customTypes.ts`:

```javascript
export interface CustomFormEvent {
  target: {
    files: File[],
    value: string, // https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement
  }
  preventDefault: () => void,
}

export type Note = {
  noteId?: string;
  content: string;
  createdAt?: string;
  attachment?: string; // Name?
  attachmentURL?: string;

};
```

To complete this code, let's add `isLoading` and `isDeleting` to the state.

<img class="code-marker" src="/assets/s.png" />So our new state and ref declarations at the top of our `Notes` component function look like this.

```javascript
const file = useRef(null);
const [note, setNote] = useState(null);
const [content, setContent] = useState("");
const [isLoading, setIsLoading] = useState(false);
const [isDeleting, setIsDeleting] = useState(false);
```

<img class="code-marker" src="/assets/s.png" />Let's also add some styles by adding the following to `src/containers/Notes.css`.

```css
.Notes form {
  padding-bottom: 15px;
}

.Notes form textarea {
  height: 300px;
  font-size: 24px;
}
```

<img class="code-marker" src="/assets/s.png" />Also, let's include the React-Bootstrap components that we are using here by adding the following to our header. And our styles, the `LoaderButton`, and the `config`.

```javascript
import { FormGroup, FormControl, ControlLabel } from "react-bootstrap";
import LoaderButton from "../components/LoaderButton";
import config from "../config";
import "./Notes.css";
```

And that's it. If you switch over to your browser, you should see the note loaded.

![Notes page loaded screenshot](/assets/notes-page-loaded.png)

Next, we'll look at saving the changes we make to our note.
