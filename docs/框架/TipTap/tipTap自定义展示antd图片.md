# 需求背景：富文本中直接 Ctrl+C 复制图片

希望实现在富文本中直接 Ctrl+C 复制图片并自动上传，如果从石墨文档或者地方粘贴过来的 HTML 格式的图片希望也能和本地上传的图片样式保持统一，即缩略图展示，支持放大操作。

# 实现

## 本地图片的复制，上传，loadIng 占位图展示及最终上传图片的展示相关逻辑的实现

TipTap 有个 FileHandler extension，专门处理文件的拖拽，复制逻辑。

```tsx
const editor = useEditor({
  extensions: [
    // 其他逻辑
    // FileHandler
    FileHandler.configure({
      allowedMimeTypes: ["image/png", "image/jpeg", "image/gif", "image/webp"],
      onPaste: (currentEditor, files, htmlContent) => {
        files.forEach((file) => {
          if (htmlContent) {
            return false;
          }
          const fileReader = new FileReader();
          fileReader.readAsDataURL(file);
          fileReader.onload = () => {
            // eslint-disable-next-line @typescript-eslint/no-use-before-define
            // 处理上传过程中占位图的展示和图片的上传
            pasteFile(file);
          };
        });
      },
    }),
  ],
});
```

上传图片的过程中展示一个 loadIng 的图片，在最开始的版本中尝试通过计算当前光标所在的位置以及 loading 图片的占位大小来实现上传过程中展示占位图，上传完毕删除占位图的效果。但是发现在富文本不同位置如文字中间，文字末尾，段落末尾，新增行开始处占位图的位置和占位大小各不相同，实现半天都不对，转而使用 AI 提供的另外一种思路去实现，详细看下方逻辑。

```tsx
const pasteFile = async (file: File) => {
  // 生成一个随机的loadIng图ID，方便后续查找
  const loadingId = generateLoadingId();
  const loadingSVG = createLoadingSVG();
  try {
    // 插入加载占位图
    editor
      ?.chain()
      .focus()
      .insertContent({
        type: "image",
        attrs: {
          src: loadingSVG,
          alt: "Loading...",
          "data-loading": loadingId,
        },
      })
      .run();

    // 调用现有上传图片方法
    const imageUrl = await uploadImage(file);
    // 先预加载下上传的图片
    await preLoadImagePromise(imageUrl || "");
    if (imageUrl) {
      // 替换为实际图片
      const imagePos = replaceLoadingImage(loadingId, imageUrl);
      // 在插入图片的后面设置光标
      if (imagePos) {
        editor.commands.setTextSelection(imagePos);
        editor.commands.focus();
      }
    } else {
      // 上传失败，删除占位图
      removeLoadingImage(loadingId);
    }
  } catch (error) {
    // ...
  }
};

// 替换loading占位图的实现
const replaceLoadingImage = (loadingId: string, imageUrl: string) => {
  if (!editor) return;
  // TipTap是基于ProseMirror的高级封装，下面的这段由AI给出，有空还是得看下ProseMirror的文档。
  const { tr } = editor.state;
  let found = false;
  let imagePos: number | undefined;
  // 编辑富文本的所有节点，返回false终止遍历
  editor.state.doc.descendants((node, pos) => {
    if (found) return false;
    // 找到了占位的loadIng图片
    if (
      node.type.name === "image" &&
      node.attrs["data-loading"] === loadingId
    ) {
      // 不计入操作栈，防止用户回撤操作出现占位图
      tr.setMeta("addToHistory", false);
      // 替换占位图节点的url,
      tr.setNodeMarkup(pos, undefined, {
        ...node.attrs,
        src: imageUrl,
        "data-loading": null, // 移除加载标识
        displayHeight: "100px", // 自定义高度
        displayWidth: "auto",
        maxWidth: "100%",
      });
      // 记录图片的终止位置
      imagePos = pos + node.nodeSize;
      found = true;
      return false;
    }
    return true;
  });
  if (found) {
    editor.view.dispatch(tr);
    return imagePos;
  }
};
```

## 正常展示缩略图，点击放大展示（直接套用 Antd 的 Image）

AntD 图片展示是使用图片的大图预览功能，不想再额外实现，这部分功能可以通过拓展 TipTap 的 Image extension 实现。当我们想要在 TipTap 中渲染 react 组件，需要使用 ReactNodeViewRenderer 这个函数将组件包裹

AI 给出的原因是：

架构差异桥接

Tiptap 基于 ProseMirror，它使用的是虚拟 DOM 和自己的节点系统，而 React 有自己的组件系统和渲染机制。ReactNodeViewRenderer 充当了两者之间的桥梁，解决了以下问题：

- 渲染系统不兼容：ProseMirror 使用原生 DOM 操作，React 使用虚拟 DOM

- 生命周期管理：需要将 ProseMirror 的节点生命周期与 React 组件生命周期同步

- 状态管理：需要在两个不同的状态管理系统之间传递数据

[文档链接]: https://tiptap.dev/docs/editor/extensions/custom-extensions/node-views/react

```tsx
import { NodeViewWrapper } from "@tiptap/react";
import { Image as AntdImage } from "antd";

// 创建图片组件
export const ImageComponent = ({ node }: { node: any }) => {
  const handleImageClick = (e: React.MouseEvent) => {
    e.stopPropagation(); // 阻止事件冒泡
  };

  // 从节点属性中获取自定义的显示尺寸，如果没有则使用默认值。这次需求全部展示固定高度，宽度根据图片比例自适应，最大不超过100%。
  const displayWidth = node.attrs.displayWidth || "auto";
  const displayHeight = node.attrs.displayHeight || "auto";
  const maxWidth = node.attrs.maxWidth || "100%";
  const maxHeight = node.attrs.maxHeight || "none";

  return (
    <NodeViewWrapper
      style={{
        display: "inline-block",
        verticalAlign: "baseline",
        padding: "0 2px",
      }}
    >
      <AntdImage
        src={node.attrs.src}
        alt={node.attrs.alt || ""}
        style={{
          height: displayHeight,
          width: displayWidth,
          maxWidth: maxWidth,
          maxHeight: maxHeight,
        }}
        onClick={handleImageClick}
      />
    </NodeViewWrapper>
  );
};
const editor = useEditor({
  extensions: [
    // 其他逻辑
    Image.configure({
      // 设置图片行内展示
      inline: true,
    }).extend({
      // 自定义渲染图片样式，
      addNodeView() {
        return ReactNodeViewRenderer(ImageComponent);
      },
      addAttributes() {
        return {
          ...this.parent?.(),
          "data-loading": {
            default: null,
            parseHTML: (element) => element.getAttribute("data-loading"),
            renderHTML: (attributes) => {
              if (!attributes["data-loading"]) {
                return {};
              }
              return {
                "data-loading": attributes["data-loading"],
              };
            },
          },
          displayWidth: {
            // ...
          },
          displayHeight: {
            // ...
          },
          maxWidth: {
            // ...
          },
          maxHeight: {
            // ...
          },
        };
      },
    }),
  ],
  // ...
});
```

## 处理粘贴 HTML 格式的情况，预期效果（支持多种图片粘贴，图文混合粘贴，单张图粘贴）

TipTap 专门提供了函数处理 HTML 的粘贴----transformPastedHTML，这里只需要返回普通的 img 标签，和赋上正确的宽高属性，Image extension 会帮我们自动处理剩下的逻辑。

```tsx
const editor = useEditor({
  editorProps: {
    // 使用 transformPastedHTML 来处理粘贴的HTML内容
    transformPastedHTML: (html: string) => {
      // 检测是否包含img标签
      if (/<img[^>]*src=["']([^"']*)["'][^>]*\/?>/gi.test(html)) {
        // 将img标签转换为自定义属性格式，保留其他HTML结构
        return html.replace(
          /<img([^>]*?)src=["']([^"']*)["']([^>]*?)\/?>/gi,
          (match, beforeSrc, src, afterSrc) => {
            const altMatch = match.match(/alt=["']([^"']*)["']/);
            const alt = altMatch ? altMatch[1] : "";
            return `<img src="${src}" alt="${alt}" data-display-height="100px" data-display-width="auto" data-max-width="100%" />`;
          }
        );
      }
      return html;
    },
  },
  // ...
});
```

这个需求的内容到此就结束了，看文档发现 TipTap 具有很强的扩展性，功能也很强大，还支持多人协作编辑，AI 集成等等强大的功能，有空可以深入研究下。
