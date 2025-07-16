# 1.BottomSheet 为什么要设计内置组件，各个的作用是什么

### 1. **手势冲突问题**

在 React Native 中，当你在 BottomSheet 内使用标准的 ScrollView、FlatList 等滚动组件时，会产生手势冲突：

- 用户向下滚动时，可能会意外触发 BottomSheet 的下拉关闭
- 用户向上滚动时，可能会意外触发 BottomSheet 的上拉展开
- 水平滚动和垂直滚动之间的优先级问题

### 2. **手势协调**

BottomSheet 需要与内部的滚动组件协调工作，决定什么时候应该滚动内容，什么时候应该移动 BottomSheet 本身。

## 各组件的具体作用

### **BottomSheetScrollView**

```typescript
// 标准 ScrollView 会有手势冲突
<ScrollView> // ❌ 会与 BottomSheet 手势冲突
  <Text>内容</Text>
</ScrollView>

// BottomSheetScrollView 解决了这个问题
<BottomSheetScrollView> // ✅ 与 BottomSheet 手势协调
  <Text>内容</Text>
</BottomSheetScrollView>
```

**特点**：

- 继承所有 `ScrollViewProps`
- 忽略 `scrollEventThrottle`、`decelerationRate`、`onScrollBeginDrag` 等影响手势的属性
- 自动处理与 BottomSheet 的手势协调

### **BottomSheetFlatList / BottomSheetFlashList**

```typescript
// 用于大量数据的高性能滚动列表
<BottomSheetFlatList
  data={tracks}
  renderItem={renderTrackItem}
  // 自动处理与 BottomSheet 的手势冲突
/>
```

**作用**：

- 解决大列表滚动与 BottomSheet 手势的冲突
- 保持列表的虚拟化性能优势
- FlashList 版本提供更好的性能

### **BottomSheetSectionList / BottomSheetVirtualizedList**

**作用**：

- `SectionList`：用于分组数据的列表（如联系人按字母分组）
- `VirtualizedList`：最底层的虚拟化列表组件，其他列表组件的基础

### **BottomSheetFooter**

```typescript
const CustomFooter = ({ animatedFooterPosition }: BottomSheetFooterProps) => {
  return (
    <BottomSheetFooter bottomInset={bottom}>
      {/* 你的 footer 内容 */}
    </BottomSheetFooter>
  );
};
```

**作用**：

- **定位管理**：自动将 footer 定位到 BottomSheet 底部
- **键盘适配**：当键盘弹出时自动调整位置
- **动画集成**：接收 `animatedFooterPosition` 参数，与 BottomSheet 动画同步
- **安全区域处理**：通过 `bottomInset` 处理设备底部安全区域

### **BottomSheetTextInput**

```typescript
<BottomSheetTextInput
  placeholder="搜索歌曲..."
  // 自动处理键盘与 BottomSheet 的交互
/>
```

**作用**：

- **键盘管理**：当 TextInput 获得焦点时，自动调整 BottomSheet 位置避免被键盘遮挡
- **焦点管理**：与 BottomSheet 的动画状态协调
- **手势处理**：确保输入时不会意外触发 BottomSheet 关闭

### **BottomSheetView**

**作用**：

- 提供与 BottomSheet 手势系统集成的基础 View 组件
- 通常用作容器组件

## 核心技术原理

### 1. **手势优先级管理**

```typescript
// 库内部的手势协调逻辑（简化版）
const gestureHandler = Gesture.Pan().onBegin(() => {
  // 判断是否应该响应手势
  if (scrollViewRef.current?.isScrolledToTop()) {
    // 允许 BottomSheet 手势
    return true;
  }
  // 否则让 ScrollView 处理
  return false;
});
```

### 2. **动画同步**

根据 [官方文档](https://gorhom.dev/react-native-bottom-sheet/components/bottomsheetscrollview)，这些组件会忽略某些会影响手势处理的属性：

- `scrollEventThrottle` - 滚动事件节流
- `decelerationRate` - 减速率
- `onScrollBeginDrag` - 开始拖拽回调

### 3. **焦点管理**

```typescript
// 支持 focusHook 用于多个滚动组件的焦点管理
<BottomSheetScrollView
  focusHook={useFocusEffect} // 来自 React Navigation
/>
```

这样设计的好处是让开发者能够**无缝使用**熟悉的 React Native 组件 API，同时自动获得与 BottomSheet 的完美集成，避免了复杂的手势冲突处理。

```typescript
// 标准 ScrollView 会有手势冲突
<ScrollView> // ❌ 会与 BottomSheet 手势冲突
  <Text>内容</Text>
</ScrollView>

// BottomSheetScrollView 解决了这个问题
<BottomSheetScrollView> // ✅ 与 BottomSheet 手势协调
  <Text>内容</Text>
</BottomSheetScrollView>
```

```typescript
// 用于大量数据的高性能滚动列表
<BottomSheetFlatList
  data={tracks}
  renderItem={renderTrackItem}
  // 自动处理与 BottomSheet 的手势冲突
/>
```

```typescript
// 库内部的手势协调逻辑（简化版）
const gestureHandler = Gesture.Pan().onBegin(() => {
  // 判断是否应该响应手势
  if (scrollViewRef.current?.isScrolledToTop()) {
    // 允许 BottomSheet 手势
    return true;
  }
  // 否则让 ScrollView 处理
  return false;
});
```
