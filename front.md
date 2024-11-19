# front

## generic button
```ts
import React from 'react';
import styles from './button.module.scss';

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
    className?: string;
};

const Button: React.FC<ButtonProps> = ({ className = '', children, ...props }) => {
    return (
        <button
            className={` ${styles.custom_button} ${className}`}
            {...props}
        >
            {children}
        </button>
    );
};

export default Button;
```

```scss
$button-bg-color: blue; // Default background color
$button-text-color: white; // Default text color

.custom_button {
    all: unset;
    display: inline-block;
    padding: 0.5rem 1rem;
    font-size: 1rem;
    font-weight: 600;
    text-align: center;
    cursor: pointer;
    background-color: $button-bg-color;
    color: $button-text-color;
    border-radius: 4px;
    transition: background-color 0.2s, color 0.2s, box-shadow 0.2s;

    &:hover {
        background-color: lighten($button-bg-color, 5%);
        color: darken($button-text-color, 10%);
    }

    &:focus {
        outline: none;
        box-shadow: 0 0 0 2px rgba(0, 0, 0, 0.2);
    }

    &:active {
        background-color: darken($button-bg-color, 10%);
        color: lighten($button-text-color, 20%);
        transform: scale(0.98);
    }

    &:disabled {
        background-color: lighten($button-bg-color, 25%);
        color: lighten($button-text-color, 40%);
        cursor: not-allowed;
    }
}
```

## icon button

- edit
- save
- trash

```ts
import React from 'react';
import styles from './icon-button.module.scss';

type IconButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  icon: string;
  color?: string;
  className?: string;
};

const IconButton: React.FC<IconButtonProps> = ({ icon, color , className = '', ...props }) => {
  return (
    <button style={ { color: color  } } className={`${styles.icon_button} ${className}`} {...props}>
      <i className={`bx ${icon}`}></i>
    </button>
  );
};

export default IconButton;
```
```scss
.icon_button {
    all: unset;
    display: inline-flex;
    justify-content: center;
    align-items: center;
    padding: 0.5rem;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s, box-shadow 0.2s;

    i {
        font-size: 1.5rem;
    }

    &:active {
        transform: scale(0.98);
    }

}
```

## dynamic field

```ts
"use client"
import React, { useEffect, useState } from 'react'
import IconButton from './icon-buttom';

interface InputFieldProps {
    edit?: () => void;
    delete?: () => void;
}

const InputField = (props: InputFieldProps) => {
    const [update, setUpdate] = useState(false);

    return (
        <div>
            <input type="text" placeholder="Enter your name" disabled={update} />
            {props.edit && <IconButton icon={`bx-${update ? 'save' : 'edit'}`} onClick={
                () => {
                    if (update) {
                        props.edit && props.edit();
                        console.log('Save');
                    }
                    setUpdate(!update);
                }
            } />}
            {props.delete && <IconButton icon='bx-trash' onClick={props.delete} />}
        </div>


    )
}

export default InputField;
```
