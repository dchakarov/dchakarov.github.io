---
title: Fitting User Content Without Clipping
date: 2025-11-18 19:25:01 Z
layout: post
image: widget-sizing/clipping.jpg
description: Adapting user-generated content to fit and look good on an iOS widget is not easy. Let's see how we can make it happen.
---

<span class="dropcap">F</span>or an app I'm working on, I needed to solve a deceptively tricky problem: how many list items can I show in a widget without clipping? The app lets users create lists - shopping, to-do, Christmas, you name it. I added a widget so they could track their current list and even tick off an item or two without opening the app. Since this is user-generated content, I control neither the number of items nor how long each one is. A shopping list might have many single-word items whilst a to-do list can have longer entries, sometimes spanning multiple lines.

## The Naive Implementation

My initial implementation didn't account for any of this. I used strict hardcoded limits depending on the widget family: 4 for small and medium, 8 for large. Here's the code:
```swift
struct ListView: View {
  @Environment(\.widgetFamily) var family
  let items: ListItem
  var itemLimit: Int {
    switch family {
    case .systemSmall:
      4
    case .systemMedium:
      4
    case .systemLarge:
      8
    default:
      0
    }
  }

  var body: some View {
    ZStack(alignment: .topLeading) {
      switch family {
      case .systemSmall,
        .systemMedium,
        .systemLarge,
        .systemExtraLarge:
        let visibleItems = Array(items[0..<itemLimit])
        VStack(alignment: .leading, spacing: 0) {
          ForEach(visibleItems, id: \.id) { (item: ListItem) in
            ItemRowView(item: item)
          }
        }
      default:
        EmptyView()
      }
    }
  }
}
```

This worked fine until I started testing with longer items. Here are a few screenshots of it "working fine" and... well... not.
![](/assets/img/widget-sizing/first-pass-small-short.png) ![](/assets/img/widget-sizing/first-pass-small-long.png)
![](/assets/img/widget-sizing/first-pass-medium-short.png)
![](/assets/img/widget-sizing/first-pass-medium-long.png)

It's harder to break with the medium or large widget, but it's still possible. Not ideal.

## Research and Solve
After some research, I discovered there's no automatic way to handle this. I needed to do the calculations myself, accounting for:
- The widget dimensions
- Padding and spacing
- Font size and line height
- How many lines each item requires

Let's apply this to the code.
```swift
var itemLimit: Int {
  itemLimitForWidgetFamily(items: items, family: family)
}

// ...

private func itemLimitForWidgetFamily(items: [ListItem], family: WidgetFamily) -> Int {
  var result = 0
  var currentHeight: CGFloat = 0

  for item in items {
    let lines = estimatedLineCount(for: item, width: widgetWidth)
    let itemHeight = lines * lineHeight
    currentHeight += itemHeight + itemSpacing
    
    if currentHeight > widgetHeight { break }
    result += 1
  }

  return result
}
```

The logic is straightforward: iterate through items, calculate each one's height based on how many lines it needs, add it to our running total, and stop when we exceed the widget's available space. The beauty of this approach? It works with your actual content, not arbitrary test data.

Notice we're referencing some unknowns: `widgetWidth`, `widgetHeight`, `lineHeight`, and `itemSpacing`. Let's resolve them.

### Widget Dimensions

We don't need to be extra precise with these; ballpark it. So let's trust the internet and use some estimated numbers here.

```swift
private func widgetHeight(for family: WidgetFamily) -> CGFloat {
  switch family {
  case .systemSmall:
    155
  case .systemMedium:
    155
  case .systemLarge:
    345
  default:
    0
  }
}

private func widgetWidth(for family: WidgetFamily) -> CGFloat {
  switch family {
  case .systemSmall:
    155
  case .systemMedium:
    329
  case .systemLarge:
    329
  default:
    0
  }
}
```

### Measuring Text: How Many Lines?

Next challenge: determining whether each item needs one line or two. I decided to cap items at 2 lines maximum - if users want to see the full text, they can open the app. This keeps the widget clean and readable.

Here's where things get interesting. We need UIKit to measure text (I know, I know):
```swift
private func estimatedLineCount(for item: ListItem, width: CGFloat) -> Int {
  stringFitsOnOneLine(item.name, width: width) ? 1 : 2
}

private func stringFitsOnOneLine(_ text: String, width: CGFloat) -> Bool {
  let constraintRect = CGSize(
    width: CGFloat.greatestFiniteMagnitude, 
    height: CGFloat.greatestFiniteMagnitude
  )
  let boundingBox = (text as NSString).boundingRect(
    with: constraintRect,
    options: [.usesLineFragmentOrigin, .usesFontLeading],
    attributes: [.font: UIFont.preferredFont(forTextStyle: .body)],
    context: nil
  )
  return boundingBox.width <= width
}
```

We're using `NSString`'s `boundingRect` method to calculate text width with the system body font. If you're using a custom font or text style, adjust accordingly.

The final piece of the puzzle is `lineHeight`, which we get from `UIFont.preferredFont(forTextStyle: .body).lineHeight`. Again, update if you're using custom typography.

## First Results

Let's test it:

![](/assets/img/widget-sizing/second-pass-small-short.png) ![](/assets/img/widget-sizing/second-pass-small-long.png)
![](/assets/img/widget-sizing/second-pass-medium-short.png)

Almost perfect! But before shipping, I ran one more test. Good thing I did.

![](/assets/img/widget-sizing/second-pass-medium-long.png)

## The Missing Piece

In the words of my favourite coding agent: *I see the real issue now!*

We forgot to account for padding around elements and the size of those checkboxes. The calculation assumed the full widget space was available for text, but that's not true. We need to subtract the UI chrome.

Here's the corrected calculation:

```swift
private func itemLimitForWidgetFamily(items: [ListItem], family: WidgetFamily) -> Int {
  var result = 0
  var currentHeight: CGFloat = 0
  let allowedWidth = widgetWidth - itemPadding
  let allowedHeight = widgetHeight - listPadding
  
  for item in items {
    let lines = estimatedLineCount(for: item, width: allowedWidth)
    let itemHeight = lines * lineHeight
    currentHeight += itemHeight + itemSpacing
    
    if currentHeight > allowedHeight { break }
    result += 1
  }
  
  return result
}
```

With this adjustment, we account for the checkbox width, the spacing between the checkbox and the text, and the horizontal padding (`itemPadding`), and the vertical padding around the list (`listPadding`). Let's verify one more time:

![](/assets/img/widget-sizing/final-pass-medium-long.png)

Perfect. No clipping, clean edges, and it works with any content length.

## Reflections

Working with widget code always feels a bit constrained. You have to accept certain limitations and adapt your expectations. But by using some lesser-known APIs and a bit of maths, we ended up in a good spot. The widget now gracefully handles user content of any length, showing as many items as will fit without breaking the visual design.

If you're building widgets with dynamic content, I hope this approach saves you some debugging time. And if you have questions or improvements to suggest, feel free to [send me a message](https://dchakarov.com/contact/).

_Header photo by <a href="https://unsplash.com/@tinkerman?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Immo Wegmann</a> on <a href="https://unsplash.com/photos/a-person-using-a-machine-to-cut-a-piece-of-paper-ASAni-6OvNM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>_
