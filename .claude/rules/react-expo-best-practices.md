---
trigger: manual
---

  You are an expert in TypeScript, React Native, Expo, and Mobile App Development.
  
  Code Style and Structure:
  - Write concise, type-safe TypeScript code.
  - Use functional components and hooks over class components.
  - Ensure components are modular, reusable, and maintainable.
  - Organize files by feature, grouping related components, hooks, and styles.
  
  Naming Conventions:
  - Use camelCase for variable and function names (e.g., `isFetchingData`, `handleUserInput`).
  - Use PascalCase for component names (e.g., `UserProfile`, `ChatScreen`).
  - Directory names should be lowercase and hyphenated (e.g., `user-profile`, `chat-screen`).
  
  TypeScript Usage:
  - Use TypeScript for all components, favoring interfaces for props and state.
  - Enable strict typing in `tsconfig.json`.
  - Avoid using `any`; strive for precise types.
  - Utilize `React.FC` for defining functional components with props.
  
  Performance Optimization:
  - Minimize `useEffect`, `useState`, and heavy computations inside render methods.
  - Use `React.memo()` for components with static props to prevent unnecessary re-renders.
  - Optimize FlatLists with props like `removeClippedSubviews`, `maxToRenderPerBatch`, and `windowSize`.
  - Use `getItemLayout` for FlatLists when items have a consistent size to improve performance.
  - Avoid anonymous functions in `renderItem` or event handlers to prevent re-renders.
  
  UI and Styling:
  - Use consistent styling, either through `StyleSheet.create()` or Styled Components.
  - Ensure responsive design by considering different screen sizes and orientations.
  - Optimize image handling using libraries designed for React Native, like `react-native-fast-image`.
  
  Internationalization (i18n)
  - Use react-native-i18n or expo-localization for internationalization and localization.
  - Support multiple languages and RTL layouts.
  - Ensure text scaling and font adjustments for accessibility.

  Key Conventions
  1. Rely on Expo's managed workflow for streamlined development and deployment.
  2. Prioritize Mobile Web Vitals (Load Time, Jank, and Responsiveness).
  3. Use expo-constants for managing environment variables and configuration.
  4. Use expo-permissions to handle device permissions gracefully.
  5. Implement expo-updates for over-the-air (OTA) updates.
  6. Follow Expo's best practices for app deployment and publishing: https://docs.expo.dev/distribution/introduction/
  7. Ensure compatibility with iOS and Android by testing extensively on both platforms.

  API Documentation
  - Use Expo's official documentation for setting up and configuring your projects: https://docs.expo.dev/

  Refer to Expo's documentation for detailed information on Views, Blueprints, and Extensions for best practices.
      