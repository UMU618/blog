---
layout: post
title: 求模版函数地址
date: 2018-03-21 18:01:32
categories: UMUTech
tags:
- dev
- cpp
- debug
---
最近用 WTL 写 Ribbon 界面，发现一个坑。

## 先看 WTL9.1 的代码

```
static void (CharFormat::*Getk_[])(IPropertyStore*) = 
{
    &CharFormat::Getk_Family, 
    &CharFormat::Getk_FontProperties_Size, 
    &CharFormat::Getk_MaskEffect<CFM_BOLD, CFE_BOLD, UI_PKEY_FontProperties_Bold>,
    &CharFormat::Getk_MaskEffect<CFM_ITALIC, CFE_ITALIC, UI_PKEY_FontProperties_Italic>,
    &CharFormat::Getk_MaskEffect<CFM_UNDERLINE, CFE_UNDERLINE, UI_PKEY_FontProperties_Underline>,
    &CharFormat::Getk_MaskEffect<CFM_STRIKEOUT, CFE_STRIKEOUT, UI_PKEY_FontProperties_Strikethrough>,
    &CharFormat::Getk_VerticalPositioning, 
    &CharFormat::Getk_Color<CFM_COLOR, UI_PKEY_FontProperties_ForegroundColor>, 
    &CharFormat::Getk_Color<CFM_BACKCOLOR, UI_PKEY_FontProperties_BackgroundColor>, 
    &CharFormat::Getk_ColorType<CFM_COLOR, CFE_AUTOCOLOR, UI_SWATCHCOLORTYPE_AUTOMATIC, UI_PKEY_FontProperties_ForegroundColorType>,
    &CharFormat::Getk_ColorType<CFM_BACKCOLOR, CFE_AUTOBACKCOLOR, UI_SWATCHCOLORTYPE_NOCOLOR, UI_PKEY_FontProperties_BackgroundColorType>,
};
```

其中 Getk_MaskEffect 是个模版函数，实现如下：

```
template <DWORD t_dwMask, DWORD t_dwEffects, REFPROPERTYKEY key>
void Getk_MaskEffect(IPropertyStore* pStore)
{
    if (SUCCEEDED(pStore->GetValue(key, &propvar)))
    {
        UIPropertyToUInt32(key, propvar, &uValue);
        if ((UI_FONTPROPERTIES)uValue != UI_FONTPROPERTIES_NOTAVAILABLE)
        {
            dwMask |= t_dwMask;
            dwEffects |= ((UI_FONTPROPERTIES) uValue == UI_FONTPROPERTIES_SET) ? t_dwEffects : 0;
        }
    }	
}
```

然后，在 VS2017 编译失败了……

> 1>X:\WTL91_5321_Final\Include\atlribbon.h(422): error C2440: 'initializing': cannot convert from 'overloaded-function' to 'void (__thiscall WTL::RibbonUI::CharFormat::* )(IPropertyStore *)'
>
> 1>X:\WTL91_5321_Final\Include\atlribbon.h(422): note: None of the functions with this name in scope match the target type

然后根据错误提示搜到：Cannot take address of template function，<https://gcc.gnu.org/bugzilla/show_bug.cgi?id=39018>，翻译一下：模版函数的地址转化，分两步走，第一步先转具化，第二步转目标类型，这样可以；直接转过去不可以！

## 再来看看 WTL10 怎么解决这个问题的！

```
static void (CharFormat::*Getk_[])(IPropertyStore*) = 
{
    &CharFormat::Getk_Family, 
    &CharFormat::Getk_FontProperties_Size, 
    &CharFormat::Getk_MaskEffectBold,
    &CharFormat::Getk_MaskEffectItalic,
    &CharFormat::Getk_MaskEffectUnderline,
    &CharFormat::Getk_MaskEffectStrikeout,
    &CharFormat::Getk_VerticalPositioning,
    &CharFormat::Getk_Color, 
    &CharFormat::Getk_ColorBack, 
    &CharFormat::Getk_ColorType,
    &CharFormat::Getk_ColorTypeBack,
};
```

原来的模版函数，已经替换成普通函数了……

```
void Getk_MaskEffectBold(IPropertyStore* pStore)
{
    Getk_MaskEffectAll(pStore, CFM_BOLD, CFE_BOLD, UI_PKEY_FontProperties_Bold);
}

void Getk_MaskEffectItalic(IPropertyStore* pStore)
{
    Getk_MaskEffectAll(pStore, CFM_ITALIC, CFE_ITALIC, UI_PKEY_FontProperties_Italic);
}

void Getk_MaskEffectUnderline(IPropertyStore* pStore)
{
    Getk_MaskEffectAll(pStore, CFM_UNDERLINE, CFE_UNDERLINE, UI_PKEY_FontProperties_Underline);
}

void Getk_MaskEffectStrikeout(IPropertyStore* pStore)
{
    Getk_MaskEffectAll(pStore, CFM_STRIKEOUT, CFE_STRIKEOUT, UI_PKEY_FontProperties_Strikethrough);
}

void Getk_MaskEffectAll(IPropertyStore* pStore, DWORD _dwMask, DWORD _dwEffects, REFPROPERTYKEY key)
{
    if (SUCCEEDED(pStore->GetValue(key, &propvar)))
    {
        UIPropertyToUInt32(key, propvar, &uValue);
        if ((UI_FONTPROPERTIES)uValue != UI_FONTPROPERTIES_NOTAVAILABLE)
        {
            dwMask |= _dwMask;
            dwEffects |= ((UI_FONTPROPERTIES)uValue == UI_FONTPROPERTIES_SET) ? _dwEffects : 0;
        }
    }
}
```
