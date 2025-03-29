# Frontend Developer Guide

This guide provides step-by-step instructions for developing and extending the frontend application. It covers the creation of new pages, components, styling, and integration with the backend API.

## Table of Contents

1. [Development Workflow](#development-workflow)
2. [Creating New Pages](#creating-new-pages)
3. [Working with Components](#working-with-components)
4. [Styling with Chakra UI](#styling-with-chakra-ui)
5. [Data Fetching and Backend Integration](#data-fetching-and-backend-integration)
6. [Form Handling](#form-handling)
7. [Theme Management](#theme-management)
8. [Troubleshooting](#troubleshooting)

## Development Workflow

Before diving into specific tasks, here's the recommended development workflow:

1. **Setup your environment**:
   ```bash
   cd frontend
   npm install
   ```

2. **Start the development server**:
   ```bash
   npm run dev
   ```

3. **Implement your changes** following the guidelines in this document

4. **Format and lint your code**:
   ```bash
   npm run lint
   ```

5. **Test your changes** by running Playwright tests:
   ```bash
   npx playwright test
   ```

6. **Build for production** to ensure your changes work in a production build:
   ```bash
   npm run build
   ```

## Creating New Pages

Creating a new page involves defining a route in TanStack Router and creating the associated component.

### 1. Define a New Route

Routes are defined in the `src/routes` directory. Each route typically has its own directory with the main component and related files.

1. **Create a new directory** for your route in `src/routes`:
   ```bash
   mkdir -p src/routes/my-new-page
   ```

2. **Create a route file** (`index.tsx` or similar):
   ```tsx
   // src/routes/my-new-page/index.tsx
   import { createFileRoute } from "@tanstack/react-router";
   
   export const Route = createFileRoute('/my-new-page/')({
     component: MyNewPage,
   });
   
   function MyNewPage() {
     return (
       <div>
         <h1>My New Page</h1>
         <p>This is my new page content.</p>
       </div>
     );
   }
   ```

3. **Add your route to the router** in your route file:
   ```tsx
   // src/routes/index.tsx
   import { Outlet, createRootRoute } from "@tanstack/react-router";
   
   // Import your new route
   import "../routes/my-new-page";
   
   export const Route = createRootRoute({
     component: () => <Outlet />,
   });
   ```

### 2. Using Route Loaders for Data Fetching

TanStack Router allows you to load data before a route is rendered using route loaders:

```tsx
// src/routes/my-new-page/index.tsx
import { createFileRoute } from "@tanstack/react-router";
import { getMyData } from "../../client/sdk.gen";

export const Route = createFileRoute('/my-new-page/')({
  component: MyNewPage,
  loader: async () => {
    // Fetch data before rendering the page
    const data = await getMyData();
    return { data };
  },
});

function MyNewPage() {
  // Access the data from the loader
  const { data } = Route.useLoaderData();
  
  return (
    <div>
      <h1>My New Page</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

### 3. Using Route Actions for Form Submissions

For handling form submissions, you can use route actions:

```tsx
// src/routes/my-new-page/index.tsx
import { createFileRoute } from "@tanstack/react-router";
import { submitMyData } from "../../client/sdk.gen";

export const Route = createFileRoute('/my-new-page/')({
  component: MyNewPage,
  action: async ({ formData }) => {
    // Handle form submission
    const result = await submitMyData(formData);
    return { result };
  },
});

function MyNewPage() {
  const navigate = useNavigate();
  const { result } = Route.useActionData() || {};
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    await Route.action({ formData });
    
    // Navigate to another route on success
    if (result?.success) {
      navigate({ to: '/' });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Working with Components

Components are the building blocks of the UI. Follow these guidelines for creating and working with components.

### 1. Create a New Component

Components should be placed in the `src/components` directory, organized by feature or function:

```bash
mkdir -p src/components/my-feature
```

Create your component file:

```tsx
// src/components/my-feature/MyComponent.tsx
import { Box, Text, Button } from "@chakra-ui/react";

interface MyComponentProps {
  title: string;
  onAction: () => void;
}

export function MyComponent({ title, onAction }: MyComponentProps) {
  return (
    <Box p={4} borderWidth="1px" borderRadius="lg">
      <Text fontSize="xl">{title}</Text>
      <Button onClick={onAction} mt={2}>
        Take Action
      </Button>
    </Box>
  );
}
```

### 2. Creating Reusable Components

For components that will be used across multiple pages, consider creating them with flexibility and reusability in mind:

```tsx
// src/components/common/Card.tsx
import { Box, BoxProps } from "@chakra-ui/react";
import { ReactNode } from "react";

interface CardProps extends BoxProps {
  children: ReactNode;
  variant?: "outline" | "filled";
}

export function Card({ children, variant = "outline", ...rest }: CardProps) {
  return (
    <Box 
      p={4} 
      borderRadius="md" 
      boxShadow={variant === "filled" ? "md" : "none"}
      borderWidth={variant === "outline" ? "1px" : 0}
      bg={variant === "filled" ? "gray.50" : "transparent"}
      {...rest}
    >
      {children}
    </Box>
  );
}
```

### 3. Using Custom Hooks for Logic

Extract reusable logic into custom hooks to keep components focused on presentation:

```tsx
// src/hooks/useMyFeature.ts
import { useState, useCallback } from "react";

export function useMyFeature(initialValue: string) {
  const [value, setValue] = useState(initialValue);
  
  const updateValue = useCallback((newValue: string) => {
    // Perform validation or transformation
    if (newValue.length > 3) {
      setValue(newValue);
    }
  }, []);
  
  return { value, updateValue };
}
```

## Styling with Chakra UI

Chakra UI is used for styling the application. It provides a set of accessible and customizable components.

### 1. Using Chakra UI Components

Use Chakra UI components instead of native HTML elements whenever possible:

```tsx
// Instead of:
<div className="container">
  <h1>Title</h1>
  <p>Content</p>
  <button>Click me</button>
</div>

// Use:
<Box p={4} maxW="container.md" mx="auto">
  <Heading as="h1" size="xl">Title</Heading>
  <Text mt={2}>Content</Text>
  <Button mt={4} colorScheme="blue">Click me</Button>
</Box>
```

### 2. Responsive Design

Utilize Chakra UI's responsive props for mobile-first design:

```tsx
<Box 
  width={{ base: "100%", md: "50%", lg: "33%" }} 
  p={{ base: 2, md: 4 }}
  fontSize={{ base: "sm", md: "md" }}
>
  Responsive content
</Box>
```

### 3. Customizing the Theme

To customize the Chakra UI theme, modify the theme configuration in `src/theme.tsx`:

```tsx
// src/theme.tsx
import { extendTheme } from "@chakra-ui/react";

const theme = extendTheme({
  colors: {
    brand: {
      50: "#f7fafc",
      100: "#edf2f7",
      // ... more colors
      900: "#1a202c",
    },
  },
  fonts: {
    heading: "'Poppins', sans-serif",
    body: "'Roboto', sans-serif",
  },
  components: {
    Button: {
      baseStyle: {
        fontWeight: "bold",
      },
      variants: {
        primary: {
          bg: "brand.500",
          color: "white",
        },
      },
    },
  },
});

export default theme;
```

### 4. Creating Custom Chakra Components

You can extend Chakra components to create custom versions:

```tsx
// src/components/common/PrimaryButton.tsx
import { Button, ButtonProps } from "@chakra-ui/react";

export function PrimaryButton(props: ButtonProps) {
  return (
    <Button 
      colorScheme="blue" 
      size="md" 
      fontWeight="bold"
      _hover={{ transform: "translateY(-2px)", boxShadow: "md" }}
      transition="all 0.2s"
      {...props} 
    />
  );
}
```

## Data Fetching and Backend Integration

Data fetching is handled using TanStack Query and the auto-generated API client.

### 1. Basic Query

Use the `useQuery` hook to fetch data:

```tsx
// src/components/users/UserList.tsx
import { useQuery } from "@tanstack/react-query";
import { UsersService } from "../../client/sdk.gen";
import { Box, Heading, List, ListItem, Spinner, Text } from "@chakra-ui/react";

export function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: () => UsersService.readUsers(),
  });
  
  if (isLoading) return <Spinner />;
  if (error) return <Text color="red.500">Error loading users: {error.message}</Text>;
  
  return (
    <Box>
      <Heading size="md" mb={4}>Users</Heading>
      <List spacing={2}>
        {data?.map(user => (
          <ListItem key={user.id} p={2} borderWidth="1px" borderRadius="md">
            {user.name} ({user.email})
          </ListItem>
        ))}
      </List>
    </Box>
  );
}
```

### 2. Mutation for Data Updates

Use the `useMutation` hook for creating, updating, or deleting data:

```tsx
// src/components/users/CreateUserForm.tsx
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { UsersService } from "../../client/sdk.gen";
import { Box, Button, FormControl, FormLabel, Input, useToast } from "@chakra-ui/react";
import { useForm } from "react-hook-form";

export function CreateUserForm() {
  const queryClient = useQueryClient();
  const toast = useToast();
  const { register, handleSubmit, reset } = useForm();
  
  const mutation = useMutation({
    mutationFn: (data) => UsersService.createUser(data),
    onSuccess: () => {
      // Invalidate queries to refetch data
      queryClient.invalidateQueries(["users"]);
      reset();
      toast({
        title: "User created",
        status: "success",
        duration: 3000,
      });
    },
    onError: (error) => {
      toast({
        title: "Failed to create user",
        description: error.message,
        status: "error",
        duration: 5000,
      });
    },
  });
  
  const onSubmit = (data) => {
    mutation.mutate(data);
  };
  
  return (
    <Box as="form" onSubmit={handleSubmit(onSubmit)} p={4} borderWidth="1px" borderRadius="md">
      <FormControl mb={4}>
        <FormLabel>Name</FormLabel>
        <Input {...register("name", { required: true })} />
      </FormControl>
      
      <FormControl mb={4}>
        <FormLabel>Email</FormLabel>
        <Input type="email" {...register("email", { required: true })} />
      </FormControl>
      
      <Button 
        type="submit" 
        colorScheme="blue" 
        isLoading={mutation.isPending}
      >
        Create User
      </Button>
    </Box>
  );
}
```

### 3. File Uploads

For file uploads, use `FormData` with axios:

```tsx
// src/components/documents/FileUpload.tsx
import { useMutation } from "@tanstack/react-query";
import { DocumentsService } from "../../client/sdk.gen";
import { Box, Button, FormControl, FormLabel, Input, useToast } from "@chakra-ui/react";
import { useState } from "react";

export function FileUpload() {
  const [file, setFile] = useState<File | null>(null);
  const toast = useToast();
  
  const mutation = useMutation({
    mutationFn: async () => {
      if (!file) return null;
      
      const formData = new FormData();
      formData.append("file", file);
      
      return DocumentsService.uploadDocument(formData);
    },
    onSuccess: () => {
      toast({
        title: "File uploaded successfully",
        status: "success",
      });
      setFile(null);
    },
  });
  
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files && e.target.files[0]) {
      setFile(e.target.files[0]);
    }
  };
  
  return (
    <Box>
      <FormControl>
        <FormLabel>Upload File</FormLabel>
        <Input type="file" onChange={handleFileChange} />
      </FormControl>
      
      <Button 
        mt={4}
        colorScheme="blue"
        isDisabled={!file}
        isLoading={mutation.isPending}
        onClick={() => mutation.mutate()}
      >
        Upload
      </Button>
    </Box>
  );
}
```

### 4. Query Optimization

Optimize queries for performance:

```tsx
// Optimized query with cacheTime and staleTime
const { data } = useQuery({
  queryKey: ["users"],
  queryFn: () => UsersService.readUsers(),
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 10 * 60 * 1000, // 10 minutes
  retry: 3,
  retryDelay: (attempt) => Math.min(attempt * 1000, 30000),
});
```

## Form Handling

Use React Hook Form for efficient form handling.

### 1. Basic Form

```tsx
// src/components/forms/BasicForm.tsx
import { useForm } from "react-hook-form";
import { Box, Button, FormControl, FormErrorMessage, FormLabel, Input, VStack } from "@chakra-ui/react";

interface FormValues {
  name: string;
  email: string;
}

export function BasicForm({ onSubmit }) {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormValues>();
  
  return (
    <Box as="form" onSubmit={handleSubmit(onSubmit)} width="100%">
      <VStack spacing={4}>
        <FormControl isInvalid={!!errors.name}>
          <FormLabel>Name</FormLabel>
          <Input
            {...register("name", {
              required: "Name is required",
              minLength: { value: 2, message: "Minimum length should be 2" },
            })}
          />
          <FormErrorMessage>{errors.name?.message}</FormErrorMessage>
        </FormControl>
        
        <FormControl isInvalid={!!errors.email}>
          <FormLabel>Email</FormLabel>
          <Input
            type="email"
            {...register("email", {
              required: "Email is required",
              pattern: {
                value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
                message: "Invalid email address",
              },
            })}
          />
          <FormErrorMessage>{errors.email?.message}</FormErrorMessage>
        </FormControl>
        
        <Button
          mt={4}
          colorScheme="blue"
          isLoading={isSubmitting}
          type="submit"
          width="full"
        >
          Submit
        </Button>
      </VStack>
    </Box>
  );
}
```

### 2. Form with Watch

Use `watch` to react to input changes:

```tsx
const { register, watch, handleSubmit } = useForm();
const password = watch("password");

// In your JSX
<FormControl>
  <FormLabel>Password</FormLabel>
  <Input type="password" {...register("password")} />
</FormControl>

<FormControl>
  <FormLabel>Confirm Password</FormLabel>
  <Input 
    type="password" 
    {...register("confirm_password", {
      validate: value => value === password || "Passwords do not match"
    })} 
  />
  <FormErrorMessage>{errors.confirm_password?.message}</FormErrorMessage>
</FormControl>
```

## Theme Management

The application supports theme switching using next-themes.

### 1. Creating a Theme Toggle

```tsx
// src/components/theme/ThemeToggle.tsx
import { Button, useColorMode } from "@chakra-ui/react";
import { MoonIcon, SunIcon } from "@chakra-ui/icons";
import { useTheme } from "next-themes";

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();
  const { colorMode, toggleColorMode } = useColorMode();
  
  const handleToggle = () => {
    toggleColorMode();
    setTheme(colorMode === "light" ? "dark" : "light");
  };
  
  return (
    <Button onClick={handleToggle} aria-label="Toggle theme">
      {colorMode === "light" ? <MoonIcon /> : <SunIcon />}
    </Button>
  );
}
```

### 2. Theme-Aware Styling

Create components that adapt to theme changes:

```tsx
// src/components/common/ThemedCard.tsx
import { Box, useColorModeValue } from "@chakra-ui/react";

export function ThemedCard({ children }) {
  const bg = useColorModeValue("white", "gray.800");
  const borderColor = useColorModeValue("gray.200", "gray.700");
  
  return (
    <Box
      p={5}
      shadow="md"
      borderWidth="1px"
      borderRadius="md"
      bg={bg}
      borderColor={borderColor}
      transition="all 0.2s"
    >
      {children}
    </Box>
  );
}
```

## Troubleshooting

Here are some common issues you might encounter during development and how to resolve them:

### 1. API Client Issues

If you encounter API client issues:

1. Ensure the backend is running
2. Regenerate the API client:
   ```bash
   npm run generate-client
   ```
3. Check browser console for CORS errors
4. Verify the API base URL in `.env` is correct

### 2. Styling Issues

For styling issues:

1. Confirm that the theme provider is properly set up in `main.tsx`
2. Check for conflicting style props on Chakra components
3. Use the React Developer Tools to inspect component props and styling

### 3. Route Issues

If routes are not working as expected:

1. Verify that your route is properly imported in the root route file
2. Check that the route path matches what you're navigating to
3. Examine TanStack Router DevTools for route errors
4. Clear browser cache or perform a hard refresh

### 4. Development Environment Issues

For development environment problems:

1. Restart the development server
2. Clear node_modules and reinstall:
   ```bash
   rm -rf node_modules
   npm install
   ```
3. Check for dependency conflicts in package.json
4. Ensure you're using the correct Node.js version specified in .nvmrc

By following these guidelines, you should be able to develop new features, pages, and components that seamlessly integrate with the existing frontend architecture.
