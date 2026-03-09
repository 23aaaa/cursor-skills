# 前端开发规范详解

## 组件拆分方法论

基于 UI、State、Event 三个维度拆分组件：

- **UI 维度**：视觉上独立的区块拆为独立组件
- **State 维度**：拥有独立状态的部分拆为独立组件
- **Event 维度**：有独立交互行为的部分拆为独立组件

## 组件架构

```
components/        # 展示组件（纯 UI，使用 shadcn/ui）
containers/        # 容器组件（负责逻辑，拼装展示组件）
hooks/             # 自定义 hooks（复杂逻辑，有状态）
utils/             # 工具函数（无状态，纯函数）
```

### 展示组件

- 只接收 props，不直接获取数据
- 使用 shadcn/ui 组件库
- 不包含业务逻辑
- 纯 UI 渲染

```tsx
// Good - 展示组件
interface UserCardProps {
  name: string;
  avatar: string;
  onEdit: () => void;
}
export const UserCard = ({ name, avatar, onEdit }: UserCardProps) => (
  <Card>
    <Avatar src={avatar} />
    <span>{name}</span>
    <Button onClick={onEdit}>Edit</Button>
  </Card>
);
```

### 容器组件

- 负责数据获取和业务逻辑
- 调用 hooks 获取数据
- 将数据传递给展示组件

```tsx
// Good - 容器组件
export const UserCardContainer = ({ userId }: { userId: string }) => {
  const { data: user } = useUser(userId);
  const { mutate: updateUser } = useUpdateUser();
  
  return <UserCard 
    name={user?.name ?? ''} 
    avatar={user?.avatar ?? ''} 
    onEdit={() => updateUser(userId)} 
  />;
};
```

## 状态管理规范

### TanStack Query（服务端状态）

用于所有需要联网的 API 数据：

```tsx
// 查询
const { data, isLoading } = useQuery({
  queryKey: ['users', userId],
  queryFn: () => fetchUser(userId),
});

// 变更
const { mutate } = useMutation({
  mutationFn: updateUser,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
});
```

### Zustand（全局客户端状态）

用于不需要联网的 UI 状态：

```tsx
const useUIStore = create<UIState>((set) => ({
  sidebarOpen: false,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
}));
```

### React Context（低频全局状态）

用于主题、用户信息等低频变化的全局状态。

### useState（组件局部状态）

用于仅在单个组件内使用的临时状态。

## 关键原则

1. 前端**不做数据处理**，数据处理放后端
2. 前端只关注 **UI 渲染**
3. 避免硬编码，使用常量或配置
4. 复杂逻辑抽取到 hooks
5. 工具函数保持无状态
