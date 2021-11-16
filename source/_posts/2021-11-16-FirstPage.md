---
title: FirstPage
date: 2021-11-16 15:07:38
tags: JavaScript
categories: JavaScript
---
# 试一下MarkDown渲染器

## 1.加个图片试试

{% asset_img 01.jpg %}

## 2.加个代码块试试

```javascript
export const getServerSideProps: GetServerSideProps<Partial<OrderRelatedProp>> = async (ctx: BaseContext) => {
	const redirect = isRedirect(ctx);
	if (redirect) {
		return redirect;
	}
	
	const DOMid = ctx?.query?.DOMid as string || "";
	const id = ctx?.query?.id as string || "";
	const baseInfo = ctx.req.baseInfo || null;
	const language = baseInfo?.language;

	const key = language === "en" ? "_Help_CMS_FAQ_EZ_SHIP_en" : `_Help_CMS_FAQ_EZ_SHIP_zh-CN`;

	const [, res] = await e(ListCmsProSubjects)([key]);
	let cmsHtml: any = null;
	if (res[key] && res[key].html) {
		cmsHtml = JSON.parse(res[key].html);
	}

	return {
		props: {
			cmsHtml,
			DOMid,
			id
		}
	};
};

export default OrderRelated;
```
## 3.试试自动构建推送
似乎可以了  补充下token

