---
name: performance-optimizer
description: Optimize React Native app performance including bridge optimization, list rendering, memory management, and profiling. Use when app is slow, laggy, or consuming excessive memory. Trigger words include "performance", "optimize", "slow", "lag", "memory", "FlatList", "rendering".
---

# Performance Optimizer

Optimize React Native app performance across rendering, bridge communication, and memory usage.

## Quick Start

Common performance issues and quick fixes:
- **Slow lists**: Use FlatList with proper optimization props
- **Bridge bottleneck**: Batch operations, use native modules
- **Memory leaks**: Clean up listeners, timers, subscriptions
- **Slow animations**: Use native driver, Reanimated library

## Instructions

### Step 1: Profile and Identify Bottlenecks

**Enable Performance Monitor:**
```javascript
// Shake device → "Show Perf Monitor"
// Or in code (dev only):
if (__DEV__) {
  require('react-native').PerformanceLogger.startTimespan('AppStartup');
}
```

**Use React DevTools Profiler:**
```bash
npm install -g react-devtools
react-devtools
```

**Use Flipper:**
- Install Flipper desktop app
- Enable React DevTools, Network, Hermes Debugger plugins
- Profile component renders and bridge calls

**Common bottlenecks:**
- Excessive re-renders
- Bridge communication overhead
- Large list rendering
- Memory leaks
- Unoptimized images

### Step 2: Optimize Component Rendering

**Use React.memo:**
```javascript
import React, { memo } from 'react';

const ExpensiveComponent = memo(({ data }) => {
  return <View>{/* Expensive rendering */}</View>;
});

// With custom comparison
const ExpensiveComponent = memo(
  ({ data }) => <View>{data.value}</View>,
  (prevProps, nextProps) => prevProps.data.id === nextProps.data.id
);
```

**Use useMemo and useCallback:**
```javascript
import { useMemo, useCallback } from 'react';

function MyComponent({ items, onPress }) {
  // Memoize expensive computations
  const sortedItems = useMemo(
    () => items.sort((a, b) => a.value - b.value),
    [items]
  );
  
  // Memoize callbacks to prevent child re-renders
  const handlePress = useCallback(
    (id) => onPress(id),
    [onPress]
  );
  
  return <List items={sortedItems} onPress={handlePress} />;
}
```

**Avoid inline functions and objects:**
```javascript
// Bad - creates new function on every render
<Button onPress={() => handlePress(item.id)} />

// Good - use useCallback
const handleItemPress = useCallback(() => handlePress(item.id), [item.id]);
<Button onPress={handleItemPress} />

// Bad - creates new object on every render
<View style={{ marginTop: 10 }} />

// Good - define outside or use StyleSheet
const styles = StyleSheet.create({
  container: { marginTop: 10 }
});
<View style={styles.container} />
```

### Step 3: Optimize List Rendering

**Use FlatList with optimization props:**
```javascript
import { FlatList } from 'react-native';

<FlatList
  data={items}
  renderItem={({ item }) => <Item data={item} />}
  keyExtractor={(item) => item.id}
  
  // Performance optimizations
  removeClippedSubviews={true}
  maxToRenderPerBatch={10}
  updateCellsBatchingPeriod={50}
  initialNumToRender={10}
  windowSize={5}
  
  // If item height is fixed
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
/>
```

**Use FlashList for better performance:**
```bash
npm install @shopify/flash-list
```

```javascript
import { FlashList } from '@shopify/flash-list';

<FlashList
  data={items}
  renderItem={({ item }) => <Item data={item} />}
  estimatedItemSize={100}
/>
```

**Optimize list items:**
```javascript
// Memoize list items
const Item = memo(({ data }) => {
  return (
    <View>
      <Text>{data.title}</Text>
    </View>
  );
});

// Use PureComponent for class components
class Item extends PureComponent {
  render() {
    return <View>{this.props.data.title}</View>;
  }
}
```

### Step 4: Optimize Bridge Communication

**Batch bridge calls:**
```javascript
// Bad - multiple bridge calls
items.forEach(item => {
  NativeModule.processItem(item);
});

// Good - single bridge call
NativeModule.processItems(items);
```

**Use InteractionManager:**
```javascript
import { InteractionManager } from 'react-native';

// Defer expensive operations until after animations
InteractionManager.runAfterInteractions(() => {
  // Expensive operation
  processData();
});
```

**Move heavy operations to native:**
```javascript
// For CPU-intensive tasks, create native module
import { NativeModules } from 'react-native';
const { HeavyComputation } = NativeModules;

const result = await HeavyComputation.process(largeDataset);
```

### Step 5: Optimize Images

**Use FastImage:**
```bash
npm install react-native-fast-image
```

```javascript
import FastImage from 'react-native-fast-image';

<FastImage
  source={{
    uri: imageUrl,
    priority: FastImage.priority.normal,
    cache: FastImage.cacheControl.immutable,
  }}
  resizeMode={FastImage.resizeMode.cover}
  style={{ width: 200, height: 200 }}
/>
```

**Optimize image sizes:**
```javascript
// Use appropriate image sizes
<Image
  source={{ uri: imageUrl }}
  style={{ width: 100, height: 100 }}
  resizeMode="cover"
/>

// Preload images
FastImage.preload([
  { uri: image1Url },
  { uri: image2Url },
]);
```

**Use WebP format:**
- Smaller file size than PNG/JPEG
- Supported on both iOS and Android
- Convert images: `cwebp input.png -o output.webp`

### Step 6: Optimize Animations

**Use native driver:**
```javascript
import { Animated } from 'react-native';

const fadeAnim = new Animated.Value(0);

Animated.timing(fadeAnim, {
  toValue: 1,
  duration: 1000,
  useNativeDriver: true,  // Enable native driver
}).start();
```

**Use Reanimated for complex animations:**
```bash
npm install react-native-reanimated
```

```javascript
import Animated, {
  useAnimatedStyle,
  withTiming,
  useSharedValue,
} from 'react-native-reanimated';

function AnimatedComponent() {
  const offset = useSharedValue(0);
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: withTiming(offset.value) }],
  }));
  
  return <Animated.View style={animatedStyle} />;
}
```

### Step 7: Fix Memory Leaks

**Clean up listeners:**
```javascript
useEffect(() => {
  const subscription = eventEmitter.addListener('event', handler);
  
  return () => {
    subscription.remove();  // Cleanup
  };
}, []);
```

**Clean up timers:**
```javascript
useEffect(() => {
  const timer = setTimeout(() => {
    // Do something
  }, 1000);
  
  return () => {
    clearTimeout(timer);  // Cleanup
  };
}, []);
```

**Clean up async operations:**
```javascript
useEffect(() => {
  let cancelled = false;
  
  async function fetchData() {
    const result = await api.fetch();
    if (!cancelled) {
      setData(result);
    }
  }
  
  fetchData();
  
  return () => {
    cancelled = true;  // Prevent state update after unmount
  };
}, []);
```

## Common Patterns

### Virtualized Lists

```javascript
import { VirtualizedList } from 'react-native';

<VirtualizedList
  data={items}
  renderItem={({ item }) => <Item data={item} />}
  keyExtractor={(item) => item.id}
  getItemCount={(data) => data.length}
  getItem={(data, index) => data[index]}
/>
```

### Lazy Loading

```javascript
import React, { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Debouncing and Throttling

```javascript
import { useCallback } from 'react';
import debounce from 'lodash/debounce';

function SearchComponent() {
  const debouncedSearch = useCallback(
    debounce((query) => {
      performSearch(query);
    }, 300),
    []
  );
  
  return (
    <TextInput
      onChangeText={debouncedSearch}
      placeholder="Search..."
    />
  );
}
```

### Pagination

```javascript
function InfiniteList() {
  const [page, setPage] = useState(1);
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const loadMore = async () => {
    if (loading) return;
    
    setLoading(true);
    const newItems = await fetchItems(page);
    setItems([...items, ...newItems]);
    setPage(page + 1);
    setLoading(false);
  };
  
  return (
    <FlatList
      data={items}
      renderItem={({ item }) => <Item data={item} />}
      onEndReached={loadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={loading ? <Loading /> : null}
    />
  );
}
```

## Profiling Tools

### React DevTools Profiler

```javascript
// Wrap components to profile
import { Profiler } from 'react';

function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  console.log(`${id} took ${actualDuration}ms to render`);
}

<Profiler id="MyComponent" onRender={onRenderCallback}>
  <MyComponent />
</Profiler>
```

### Performance Monitoring

```javascript
import { PerformanceObserver, performance } from 'react-native-performance';

const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});

observer.observe({ entryTypes: ['measure'] });

performance.mark('start');
// ... operation
performance.mark('end');
performance.measure('operation', 'start', 'end');
```

## Troubleshooting

**App feels sluggish:**
- Profile with React DevTools
- Check for excessive re-renders
- Optimize list rendering
- Use native driver for animations

**High memory usage:**
- Check for memory leaks (listeners, timers)
- Optimize image sizes
- Implement pagination for large lists
- Profile with Xcode Instruments (iOS) or Android Profiler

**Slow startup:**
- Reduce initial bundle size
- Lazy load components
- Optimize native module initialization
- Use Hermes engine

**Janky animations:**
- Use native driver
- Avoid bridge calls during animations
- Use Reanimated for complex animations
- Reduce layout complexity

## Best Practices

1. **Profile first**: Identify actual bottlenecks before optimizing
2. **Memoize appropriately**: Don't over-optimize, profile to confirm
3. **Optimize lists**: Use FlatList/FlashList with proper props
4. **Native driver**: Always use for animations when possible
5. **Clean up**: Remove listeners, timers, subscriptions
6. **Batch operations**: Minimize bridge calls
7. **Lazy load**: Load components and data on-demand
8. **Monitor**: Track performance metrics in production
