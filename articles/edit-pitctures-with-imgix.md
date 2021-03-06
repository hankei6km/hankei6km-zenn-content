---
id: edit-pitctures-with-imgix
title: imgix ใๅฅฝใ
emoji: ๐ผ๏ธ
type: idea
topics:
  - imgix
  - image
published: true
---

็ช็ถใงใใ [imgix](https://imgix.com/) ใๅฅฝใใงใใ

Headless CMS ใใ [Rendering API](https://docs.imgix.com/apis/rendering) ใฎใฟใฎๅฉ็จใงใใ[^cms-imgix]ใ[ใใฉใกใผใฟใผใไฝๆใใใฆใงใใขใใช](https://github.com/hankei6km/image-url-workbench)ใใ[WYSIWYG ใจใใฃใฟใผใใใใฉใกใผใฟใผใๆๅฎใใใใผใซ](https://hankei6km.github.io/rehype-image-salt-doc/)ใใไฝใใใใใซๅฅฝใใงใใ

ไปฅๅใซใใ[imgix Rendering API Showcase](https://hankei6km.github.io/mardock/deck/try-imgix-rendering-api)ใในใฉใคใใไฝใฃใใใใใฎใงใใใใใฃใใ Zenn ใๅงใใใฎใงใใฎ่พบใใใๅฐใๆธใใฆใฟใใใใจใ

ใชใใไปๅใฏ imgix ใไฝฟใฃใ(ๅไบบใใญใฐใชใฉใงใฎ)็ปๅใฎๅ ๅทฅใใจใใจใใใชใๆธใใฆใใใ ใใงใใ็ปๅใฎๆ้ฉๅ(ๅไธๅ)ใชใฉใฎๅฎ็จ็ใชๆๅ ฑใฏไปฅไธใฎ่จไบใ่ฉณใใ่งฃ่ชฌใใใฆใใพใใ

@[card](https://zenn.dev/manalink/articles/manalink-imgix)

[^cms-imgix]: [microCMS](https://microcms.io/) ใจ [Prismic](https://prismic.io/) ใ็ปๅใใใฏใจใณใใซ imgix ใๅฉ็จใใฆใใพใใ

## ใฉใใๅฅฝใ

[ใธใงใฎใณใฐใฎใกใข](https://hankei6km.github.io/mardock/deck/category/jog)ใไฝใฃใฆใใใฎใงใใใๅ็ใฏในใใใงใชใใจใชใๆฎๅฝฑใใฆใใใ็ถๆใงใใใชใฎใงๅ็ใๅฉ็จใใใจใใซใใกใใฃใจใ ใไฟฎๆญฃใใใใ็ใชใใจใ็บ็ใใพใใ

ใใใชใจใใซ imgix ใใใใใใๆๅฉใใใฆใใใพใใ

ใพใใไปๆใฏ [nuxt-image](https://image.nuxtjs.org/) ใชใฉใใใใฎใงๆ่ญใใใใจใฏๅฐใชใใงใใใ1 ๆใฎ็ปๅใใใฏใจใชใผใใฉใกใผใฟใผใ ใใง่คๆฐใตใคใบใฎ็ปๅใ็ๆใงใใใ็นใใใใใใใงใใ

## ็ปๅใใกใใฃใจ่ชฟๆดใใใ

ไปฅไธใฏใๆใฃใฆใใๅ็ใจใกใใฃใจ้ใใใจใใใจใใซไฝฟใฃใฆใใใใฉใกใผใฟใผใๆๅฎใใใจใใฎใตใณใใซใงใใ

:::message
ใใฎ่จไบๅจไฝใง `auto=compress,format` ใจใตใคใบๆๅฎใใใใฆใใใฎใงใชใชใธใใซ็ปๅใ็ป่ณชใฏ่ฅๅนฒไฝไธใใฆใใพใใ
:::

### Automatic ใง่ฏใใใซใใ

[Automatic](https://docs.imgix.com/apis/rendering/auto) ใฏไธปใซ็ปๅใๅง็ธฎใใใใใฉใผใใใใ็ฐๅขใซใใใใฆ้ธๆใใใใใจใซไฝฟใใใพใใใไปใซใ [`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) ใชใฉใใใใพใใ

[`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) ใฏๅคง้ๆใซใใใจใ็ปๅใ่ฏใใใซใใใๆๅฎใงใใRendering API ใซใฏ [Adjustment](https://docs.imgix.com/apis/rendering/adjustment) ใชใฉใใใใพใใใใใฏใใใงๆ้ใใใใใพใใ

ใใกใใฃใจๆใใฎๅ็ใซใชใฃใฆใใพใฃใใใจใใใจใใชใฉใซ [`enhance`](https://docs.imgix.com/apis/rendering/auto/auto#enhance) ใชใๆๅฎใใใ ใใงๅคงไฝ่ฏใใใช็ปๅใซใชใใพใ(ๅไบบใฎๆๆณใงใ)ใ

![enhance ็จใฎ็ปๅ(ใชใชใธใใซ)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62f516d903e540a5a09243f905acc411/edit-pitctures-with-imgix-enhance.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![enhance ็จใฎ็ปๅ(ใใฉใกใผใฟใผๆๅฎ)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/62f516d903e540a5a09243f905acc411/edit-pitctures-with-imgix-enhance.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance)*auto=enhance*

![enhance ็จใฎ็ปๅ(ใชใชใธใใซ)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![enhance ็จใฎ็ปๅ(ใชใชใธใใซ)](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance)*auto=enhance*

### Rotation ใงๅพใใไฟฎๆญฃใใ

่ชญใใงๅญใฎใใจใใงใใใใกใใฃใจๅพใใฆใใพใฃใใใจใใใจใใซ [`rot`](https://docs.imgix.com/apis/rendering/rotation/rot) ใไฝฟใฃใฆใใพใใไปฅไธใฏใใใใใใๅ่ปขใใใฆใใพใใ

![Rotation ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a661a61f54ec4b5288333cce3e5d3949/edit-pitctures-with-imgix-rotation.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![Rotation ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/a661a61f54ec4b5288333cce3e5d3949/edit-pitctures-with-imgix-rotation.jpg?w=600\&h=300\&rot=340)*rot=340*

### Focal Point Crop ใงไปปๆใฎไฝ็ฝฎใๅใๅบใ(ใบใผใ ใใ)

ใธใงใฎใณใฐใฎใกใขใงใฏๅ็ใ่ๆฏ็ปๅใซใใฆใใญในใใใใถใใฆใใใฎใงใๅปบ็ฉใชใฉใ้ใชใฃใฆใใญในใใ่ชญใฟใซใใใใจใใจใใใใพใซใใใพใใใใใชใจใใซๅฉ็จใใฆใใพใ[^rect]ใ

ใใใฏๆ้ ใใกใใฃใจใใใใซใใใฎใงใใใไปฅไธใฎใใใชๆใใงๆๅฎใใพใใ

[Size](https://docs.imgix.com/apis/rendering/size) ๆๅฎใง [Fit mode](https://docs.imgix.com/apis/rendering/size/fit) ใ [`crop`](https://docs.imgix.com/apis/rendering/size/fit#crop)ใ[Crop Mode](https://docs.imgix.com/apis/rendering/size/crop) ใ [`focalpoint`](https://docs.imgix.com/apis/rendering/size/crop#focalpoint) ใซใใพใ(ใใฎๆ็นใง็ปๅใฏๅคๅใใชใ)ใ

![focalpoint ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint)*fit=crop\&crop=focalpoint*

[Focal Point Zoom](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-z) ใงใบใผใ ใใพใใ

![focalpoint ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint\&fp-z=1.8)*fit=crop\&crop=focalpoint\&fp-z=1.8*

[Focal Point X Position](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-x) ใจ [Focal Point Y Position](https://docs.imgix.com/apis/rendering/focalpoint-crop/fp-y) ใงใบใผใ ใใ้ ๅใๅคๆดใใพใใ

![focalpoint ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/f62544aa8a2d419dabb9ff52208405d6/edit-pitctures-with-imgix-focalpoint.jpg?w=600\&h=300\&auto=compress%2Cformat%2Cenhance\&fit=crop\&crop=focalpoint\&fp-z=1.8\&fp-x=0.68\&fp-y=0.55)*fit=crop\&crop=focalpoint\&fp-z=1.8\&fp-x=0.68\&fp-y=0.55*

:::message
็ปๅใๅใๅบใใฆ่กจ็คบใใๅพใใตใผใใผไธใซใฏใชใชใธใใซใฎใใกใคใซใๆฎใฃใฆใใพใใ

ใใฉใกใผใฟใผใ้คๅปใใฆ่กจ็คบใใใฐใชใชใธใใซใฎ็ปๅใ่กจ็คบใงใใใฎใงๆณจๆใใฆใใ ใใใ
:::

[^rect]: ไผผใใใใชใใจใฏ [Source Rectangle Region](https://docs.imgix.com/apis/rendering/size/rect) ใงใใงใใพใใ

## ใใฃใซใฟใผ็ใซ่ฆใ็ฎใๅคๆดใใ

ไปฅไธใฏใimgix ใ ใจใใใชใใจใใงใใใใใจใใใทใงใผใฑใผใน็ใชใใฉใกใผใฟใผใฎๆๅฎใงใใ

ใCSS ใงๆๅฎใงใใใ้ ็ฎใใใใพใใใblur ใชใฉใฏ imgix ๅดใงๅ ๅทฅใใฆใใใจใ่ปข้้ใๆธใใใใใจใใๅฉ็นใใใใพใใ

### blur

![blur ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/abb6bbc126ec4e62a4dd5dfb2d6c0d11/edit-pitctures-with-imgixi-blur.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![blur ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/abb6bbc126ec4e62a4dd5dfb2d6c0d11/edit-pitctures-with-imgixi-blur.jpg?w=600\&h=300\&blur=50)*blur=50*

### hue shift

![blur ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c530685f5e2945c583ad41322600fc49/edit-pitctures-with-imgix-hue-shift.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![blur ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/c530685f5e2945c583ad41322600fc49/edit-pitctures-with-imgix-hue-shift.jpg?w=600\&h=300\&hue=316)*hue=316*

### vibrance

![vibrance ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea854b9a431440c7a585b3cdd8bea587/edit-pitctures-with-imgix-vibrance.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![vibrance ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/ea854b9a431440c7a585b3cdd8bea587/edit-pitctures-with-imgix-vibrance.jpg?w=600\&h=300\&vib=100)*vib=100*

### sepia tone

![sepia tone ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1a3ee1b2642d4fd9b729f1ddb9d6b8ec/edit-pitctures-with-imgix-sepia-tone.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![sepia tone ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/1a3ee1b2642d4fd9b729f1ddb9d6b8ec/edit-pitctures-with-imgix-sepia-tone.jpg?w=600\&h=300\&sepia=80)*sepia=80*

### monochrome

![monochrome ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7705ec3d5d804ffa903a7bb97f244168/edit-pitctures-with-imgix-monochrome.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![monochrome ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/7705ec3d5d804ffa903a7bb97f244168/edit-pitctures-with-imgix-monochrome.jpg?w=600\&h=300\&monochrome=ff4a4a4a)*monochrome=ff4a4a4a*

### duo tone

![duo tone ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3fd1988e07af41fa98a1d7abf3287564/edit-pitctures-with-imgix-duo-tone.jpg?w=600\&h=300\&auto=compress%2Cformat)*ใชใชใธใใซ*

![duo tone ็จใฎ็ปๅ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/3fd1988e07af41fa98a1d7abf3287564/edit-pitctures-with-imgix-duo-tone.jpg?w=600\&h=300\&duotone=000080%2CFA8072\&duotone-alpha=100)*duotone=000080,FA8072\&duotone-alpha=100*

## ใใฉใกใผใฟใผใไฝใใฎใฏๅคงๅคใชใฎใงใฏ

ใฏใจใชใผใใฉใกใผใฟใผใ ใใง็ปๅใๅ ๅทฅใงใใใฎใ Rendering API ใฎ่ฏใใงใใใๅ็ใใจใซๆๅใงใใฉใกใผใฟใผใไฝใใฎใฏใใฏใๅคงๅคใงใใ

### ImageURL Workbench

ใใใชใจใใฎใใใซใ[ImageURL Workbench](https://image-url-workbench.vercel.app/)ใใไฝๆใใพใใ(ๅฎฃไผ)ใ

[![ImageURL Workbench ใงใใฉใกใผใฟใผใๆๅฎใ็ปๅใใใฌใใฅใผใใฆใใในใฏใชใผใณใทใงใใ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?w=600\&h=383\&auto=compress%2Cformat)*ImageURL Workbench*](https://image-url-workbench.vercel.app/)

@[card](https://image-url-workbench.vercel.app/)

ใใฌใใฅใผใ็ปๅใตใคใบใชใฉใ็ขบ่ชใใชใใใใฉใกใผใฟใผใๆๅฎใงใใพใใ

![ImageURL Workbench ใง็ปๅใฎใใฌใใฅใผใใตใคใบใ่กจ็คบใใฆใใในใฏใชใผใณใทใงใใ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?auto=compress%2Cformat\&w=600\&h=430\&rect=40%2C80%2C600%2C430\&border=1%2C55000000\&border-radius=1)*ใใฌใใฅใผใจใตใคใบ่กจ็คบ*

![ImageURL Workbench ใงใใฉใกใผใฟใผใๆๅฎใใฆใใ้ ๅใฎในใฏใชใผใณใทใงใใ](https://images.microcms-assets.io/assets/1fff6177c5c74aac8d5158dc17492c92/959ee4f71196411d9c01994f799ac703/edit-pitctures-with-imgix-image-url-workbench.png?auto=compress%2Cformat\&w=600\&h=600\&rect=650%2C130%2C600%2C600\&border=1%2C55000000\&border-radius=1)*ใใฉใกใผใฟใผใๆๅฎ*

### ใฌในใใณในๅฏพๅฟใฎใฟใฐใไฝใใใข

@[youtube](Nj6RsEriwzQ)

### ้กๆคๅบใๅฉ็จใใใใข

@[youtube](p6C0qZHndz8)

## ใใใใซ

ๅ็ใใจใซใฏใจใชใผใใฉใกใผใฟใผใงใใใใ็ทจ้ใใใฎใฏๆ้ใงใใใใใใผใงใฎๅฉ็จใชใใใใใใฎใๆฅฝใใฟใฎ 1 ใคใชใฎใใชใจใ

ใจใใๆใใงใใใฃใฑใ imgix ใๅฅฝใใใๅ็ขบ่ชใใ่จไบใงใใใ

ใชใใใใฎ่จไบใงใฏๅ็ใฎๅ ๅทฅใซใคใใฆๆธใใพใใใใimgix ใฏในใฏใชใผใณใทใงใใใฎๅ ๅทฅใงใไพฟๅฉใซๅฉ็จใงใใใฎใง[ๅฅ่จไบใซใใฎ่พบใฎใใจใๆธใใพใใ](https://zenn.dev/hankei6km/articles/process-screen-shot-by-imgix)ใ
