## Spring

```java
package com.siemens.trail.utilites;

import lombok.Getter;
import lombok.Setter;
import org.springframework.web.ErrorResponse;

@Getter @Setter
public class APIResponse<T> {
    boolean success;
    T data;
    String message;
    int statusCode;

    public APIResponse(T data) {
        success = true;
        statusCode = 200;
        this.data = data;
    }

    public APIResponse(String message, int statusCode) {
        success = false;
        this.message = message;
        this.statusCode = statusCode;
    }
}
```
usage

return type: **ResponseEntity<APIResponse<String>>**
```java
if (randomDouble < 0.5) {
    return ResponseEntity.ok().body(new APIResponse<>("Request Failed",400)) ;
}else{
    return ResponseEntity.ok().body(new APIResponse<>("The request was successful!"));
}
```

## next

```typescript 
"use client"
import React, { createContext, useContext, ReactNode } from 'react';

interface FetchProviderProps {
    children: ReactNode;
}

interface FetchContextType {
    get: (url: string) => Promise<any>;
    post: (url: string, data: any) => Promise<any>;
    put: (url: string, data: any) => Promise<any>;
    delete: (url: string) => Promise<any>;
}

const FetchContext = createContext<FetchContextType | undefined>(undefined);

export const FetchProvider = ({ children }: FetchProviderProps) => {
    const fetchWrapper = async (url: string, options: RequestInit) => {
        const response = await fetch(url, options);
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        return response.json();
    };

    const get = (url: string) => fetchWrapper(url, { method: 'GET' });
    const post = (url: string, data: any) => fetchWrapper(url, { method: 'POST', body: JSON.stringify(data), headers: { 'Content-Type': 'application/json' } });
    const put = (url: string, data: any) => fetchWrapper(url, { method: 'PUT', body: JSON.stringify(data), headers: { 'Content-Type': 'application/json' } });
    const deleteRequest = (url: string) => fetchWrapper(url, { method: 'DELETE' });

    return (
        <FetchContext.Provider value={{ get, post, put, delete: deleteRequest }}>
            {children}
        </FetchContext.Provider>
    );
};

export const useFetch = () => {
    const context = useContext(FetchContext);
    if (context === undefined) {
        throw new Error('useFetch must be used within a FetchProvider');
    }
    return context;
};
```
usage

```typescript
const { get } = useFetch();

  const fetchData = async () => {
    const response = await get('http://localhost:8080/users/error');
    if (response.success) {
      setError(null);
      setData(response.data);
    }else{
      setData({});
      setError(response.message);
    }
  }
```

