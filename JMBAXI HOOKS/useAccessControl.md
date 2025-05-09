# useAccessControl Hook

The `useAccessControl` hook helps you manage role-based permissions in the UI. It checks whether a user has access to a specific action, feature, or page based on their assigned permissions.

## Why Use It?

Instead of manually checking permission codes in your components, you can use `checkAccess()` to:

- Show or hide UI elements
- Enable/disable buttons
- Restrict routes or pages
- Apply logic based on the current user's access level

---

# How It Works

```js
import { find, some } from "lodash";
import usePermissions from "./usePermissions";
import { PERMISSIONS } from "@/utilities/permissions";

const useAccessControl = () => {
  const { permissions: grantedPermissions } = usePermissions();

  const checkAccess = (permission) => {
    if (permission === PERMISSIONS.PUBLIC) {
      return true;
    }
    if (Array.isArray(permission)) {
      return some(
        permission,
        (permissionElement) =>
          !!find(grantedPermissions, ({ code }) => permissionElement === code),
      );
    }
    return !!find(grantedPermissions, ({ code }) => permission === code);
  };

  return { checkAccess, grantedPermissions };
};

export default useAccessControl;
```

---

### Parameters

- `permission`: Can be a single string (like `PASSENGER.GET_PASSENGERS`), or an array of strings. Also accepts `PERMISSIONS.PUBLIC` as a special case to always allow access.

---

### Returns

- `checkAccess(permission)`: Returns `true` if the user has at least one of the given permissions.
- `grantedPermissions`: The full list of permission codes available to the current user.

---
### usePermissions Hook

The `usePermissions` hook provides access to the current user's granted permissions and also allows updating them within the application state. Permissions can be updated in the atom by setting them when fetched, either from an API response or extracted from the user's ID token.

```js
import { currentUserPermissions } from "@/store";
import { useAtomValue, useSetAtom } from "jotai";

const usePermissions = () => {
  const permissions = useAtomValue(currentUserPermissions);
  const setPermissions = useSetAtom(currentUserPermissions);

  return {
    permissions,
    setPermissions,
  };
};

export default usePermissions;
```

---

### Parameters

- `permission`: Can be a single string (like `FINANCE.GET_FINANCES`), or an array of strings. Also accepts `PERMISSIONS.PUBLIC` as a special case to always allow access.
# Example Usage

```jsx
import useAccessControl from "@/hooks/useAccessControl";
import { PERMISSIONS } from "@/utilities/permissions";

const FinanceSection = () => {
  const { checkAccess } = useAccessControl();

  if (!checkAccess(PERMISSIONS.PASSENGERS.GET_PASSENGERS)) {
    return null; // Hide the section if access is denied
  }

  return (
    <div>
      <h2>Finance Dashboard</h2>
      {/* secure content here */}
    </div>
  );
};
```

---

# Sample Permissions

This is an example of how permissions may be structured:

```js
export const PERMISSIONS = {
  PASSENGERS: {
    GET_PASSENGERS: "GET_PASSENGERS",
    GET_PASSENGER: "GET_PASSENGER",
    ESCORT_PASSENGER: "ESCORT_PASSENGER",
    UPDATE_PHOTO_PASSENGER: "UPDATE_PHOTO_PASSENGER",
    REGULARIZE_TERMINAL_MOVEMENT: "REGULARIZE_TERMINAL_MOVEMENT",
  },
  SHIPS: {
    GET_SHIPS: "GET_SHIPS",
    GET_SHIP: "GET_SHIP",
  },
};
```

You can extend this object to reflect the permissions structure relevant to your application.
