# WebSocketProvider Component

## Description

The `WebSocketProvider` component is a context provider that manages the WebSocket connection and its interactions within a React application. It connects to a WebSocket server and provides the necessary data (counts, notifications, connection status, etc.) to its children via the `WebSocketContext`. This allows for real-time updates in the application while ensuring that the WebSocket connection is properly handled and synchronized with the authentication state.

## How It Works

1. **WebSocket Initialization**: 
   - The component creates a WebSocket URL using the `jwtToken`, `userRoles`, and `userId` from the authenticated session. The WebSocket connection is established once the user is signed in.
   
2. **WebSocket Connection**: 
   - The `useSocketConnection` hook from `react-use-websocket` is used to establish and manage the WebSocket connection. The connection status is monitored and updated based on the WebSocket's ready state.

3. **Data Updates**:
   - When new messages are received from the WebSocket (`lastMessage`), the component processes the data, updating various state values such as `counts`, `notifications`, etc., based on the action type in the received data.

4. **Context Provider**:
   - The component wraps its children with `WebSocketContext.Provider`, making the WebSocket data and utility functions available to the rest of the application. The context provides values such as connection status, counts, notifications, functions to send messages, disconnect the WebSocket, etc.

## Code

```javascript
import { WebSocketContext } from "@/contexts/WebSocketContext";
import useAuth from "@/hooks/useAuth";
import useGlobalAlert from "@/hooks/useGlobalAlert";
import config from "@/utilities/config.json";
import {
  AUTH_STATES,
  COMMON_MESSAGES,
  GLOBAL_ALERT_TYPES,
  WEBSOCKET_ACTION_TYPES,
  WEBSOCKET_CONNECTION_STATUS,
} from "@/utilities/constants";
import { generateQueryString } from "@/utilities/query";
import PropTypes from "prop-types";
import { useCallback, useEffect, useMemo, useState } from "react";
import {
  ReadyState,
  default as useSocketConnection,
} from "react-use-websocket";

const WebSocketProvider = ({ children }) => {
  const [counts, setCounts] = useState({
    approvalCount: 0,
    manifestCount: 0,
  });
  const [notifications, setNotifications] = useState({});
  const [notificationCount, setNotificationCount] = useState(0);
  const [terminalTrafficData, setTerminalTrafficData] = useState({});
  const [connectionStatus, setConnectionStatus] = useState(
    WEBSOCKET_CONNECTION_STATUS.CLOSED,
  );
  const { logOut } = useAuth();
  const [webSocketUrl, setWebSocketUrl] = useState();
  const { pushAlert } = useGlobalAlert();

  const { user, authState, getCurrentSession } = useAuth();
  const { userId = "", userRoles = "" } = user || {};

  const createWebSocketUrl = async () => {
    const { jwtToken } = await getCurrentSession();
    return `${config.websocketURL}?${generateQueryString({
      authorization: jwtToken,
      role: userRoles,
      "user-id": userId,
    })}`;
  };

  useEffect(() => {
    if (authState === AUTH_STATES.SIGNED_IN) {
      createWebSocketUrl().then((signedUrl) => {
        setWebSocketUrl(signedUrl);
      });
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [userId, userRoles, authState]);

  const { lastMessage, readyState, getWebSocket, sendMessage } =
    useSocketConnection(
      webSocketUrl,
      {
        shouldReconnect: () => authState === AUTH_STATES.SIGNED_IN || false,
      },
      authState === AUTH_STATES.SIGNED_IN && !!webSocketUrl,
    );

  useEffect(() => {
    const currentConnectionStatus = ReadyState[readyState];
    setConnectionStatus(currentConnectionStatus);
  }, [readyState]);

  const updateCountsData = (data) => {
    setCounts(data);
  };

  const updateNotificationsData = (data) => {
    setNotifications(data);
    setNotificationCount((prev) => prev + 1);
  };

  const updateTerminalTrafficData = (data) => {
    setTerminalTrafficData(data);
  };

  useEffect(() => {
    if (lastMessage?.data) {
      const updatedData = JSON.parse(lastMessage?.data);
      if (updatedData?.type == WEBSOCKET_ACTION_TYPES.COUNT) {
        updateCountsData(updatedData?.counts);
      }
      if (updatedData?.type === WEBSOCKET_ACTION_TYPES.NOTIFICATION) {
        updateNotificationsData(updatedData?.notification);
      }
      if (updatedData?.type === WEBSOCKET_ACTION_TYPES.TERMINAL_TRAFFIC) {
        updateTerminalTrafficData(updatedData?.terminalTraffic);
      }
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [lastMessage]);

  const handleClickSendMessage = useCallback(
    (route, data) => {
      sendMessage(JSON.stringify({ action: route, ...data }));
    },
    [sendMessage],
  );

  const disconnectWebSocket = useCallback(() => {
    const socket = getWebSocket();
    if (socket) {
      socket.close();
    }
  }, [getWebSocket]);

  useEffect(() => {
    if (
      authState === AUTH_STATES.SIGNED_OUT &&
      ReadyState[readyState] === WEBSOCKET_CONNECTION_STATUS.OPEN
    ) {
      disconnectWebSocket();
    }
  }, [authState, disconnectWebSocket, readyState]);

  const memoValues = useMemo(
    () => ({
      connectionStatus,
      counts,
      data: lastMessage,
      disconnectWebSocket,
      handleClickSendMessage,
      notificationCount,
      notifications,
      setCounts,
      setNotificationCount,
      setNotifications,
      setTerminalTrafficData,
      terminalTrafficData,
      updateTerminalTrafficData,
    }),
    [
      connectionStatus,
      counts,
      disconnectWebSocket,
      handleClickSendMessage,
      lastMessage,
      notifications,
      notificationCount,
      terminalTrafficData,
    ],
  );

  return (
    <WebSocketContext.Provider value={memoValues}>
      {children}
    </WebSocketContext.Provider>
  );
};

WebSocketProvider.propTypes = {
  children: PropTypes.node.isRequired,
};

export default WebSocketProvider;
```
# Key Concepts

### Context Creation
The `WebSocketContext` is used to create a context provider that makes WebSocket data and functions accessible to all child components.

### WebSocket Connection
The component establishes a WebSocket connection using the `useSocketConnection` hook, which is configured to reconnect if necessary. The connection's status is monitored and updated accordingly.

### Message Processing
The `lastMessage` received from the WebSocket is checked to determine the type of data it contains. The component processes this data based on the action type (e.g., `COUNT`, `NOTIFICATION`, `TERMINAL_TRAFFIC`, `REGISTER_SESSION`) and updates the corresponding state.

## Usage Example

To use the `WebSocketProvider` and send messages via the WebSocket connection, you can follow this example:

```javascript
import { useEffect } from "react";
import { useWebSocket } from "@/hooks/useWebSocket";
import { WEBSOCKET_CONNECTION_STATUS, WEBSOCKET_CONNECTION_ROUTES } from "@/utilities/constants";

const ExampleComponent = ({ user }) => {
  const websocket = useWebSocket();

  useEffect(() => {
    // Prepare data to send
    const messageData = {
      // Include relevant data here
    };

    // Check if WebSocket is open before sending a message
    if (websocket.connectionStatus === WEBSOCKET_CONNECTION_STATUS.OPEN) {
      websocket.handleClickSendMessage(
        WEBSOCKET_CONNECTION_ROUTES.SOME_ROUTE, // Use the appropriate route
        messageData
      );
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [websocket.connectionStatus, user]); // Re-run when connectionStatus or user changes

  return (
    <div>
      {/* Render component content */}
    </div>
  );
};
```
## Dependencies

Below are the dependencies used in the WebSocketProvider component:

### Internal Dependencies
- **`useWebSocket`**: A context-based custom hook. It is used to interact with the WebSocket context, allowing access to WebSocket functions (e.g., sending messages) and state (e.g., connection status, received data).
- **`useGlobalAlert`**: A custom hook to trigger global alerts (e.g., success, error) across the application.
- **`WEBSOCKET_CONNECTION_STATUS`**: A constant that represents various WebSocket connection states (e.g., OPEN, CLOSED).
- **`WEBSOCKET_CONNECTION_ROUTES`**: A constant object containing predefined WebSocket message routes used for sending specific types of messages.
- **`WEBSOCKET_ACTION_TYPES`**: Constants representing different WebSocket action types (e.g., COUNT, NOTIFICATION).
- **`AUTH_STATES`**: Constants representing various authentication states (e.g., SIGNED_IN, SIGNED_OUT).
- **`COMMON_MESSAGES`**: Predefined messages used throughout the application for consistent communication (e.g., error or success messages).
- **`GLOBAL_ALERT_TYPES`**: Constants defining types of global alerts (e.g., SUCCESS, ERROR).
- **`generateDeviceID`**: A utility function that generates a unique device ID.
- **`config`**: Contains application configuration values like WebSocket URLs and Cognito User Pool details.

### External Dependencies
- **`useEffect`**: A React hook to perform side effects in functional components.
- **`localStorage`**: Browser's local storage used to retrieve saved tokens.

### Example of Replacing External Dependencies
Replace external dependencies like `useEffect` and `localStorage` with mock comments or placeholders if you are writing mock implementations or need alternative approaches for environments that do not support them.

```javascript
// Replace with your effect management logic (e.g., custom hooks or alternatives)
useEffect(() => {
  // Your logic here
}, [dependencies]);

// Replace with alternative storage or mock implementation
const refreshToken = localStorage.getItem("your-key");
```


