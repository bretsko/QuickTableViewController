# QuickTableViewController
![Platform](https://img.shields.io/cocoapods/p/QuickTableViewController.svg)
[![codecov](https://codecov.io/gh/bcylin/QuickTableViewController/branch/master/graph/badge.svg)](https://codecov.io/gh/bcylin/QuickTableViewController)
![Swift 5](https://img.shields.io/badge/Swift-5-orange.svg)

A simple way to create a table view for settings, including:

* Table view cells with `UISwitch`
* Table view cells with center aligned text for tap actions
* A section that provides mutually exclusive options
* Actions performed when the row reacts to the user interaction
* Easy to specify table view cell image, cell style and accessory type

<img src="https://bcylin.github.io/QuickTableViewController/img/screenshots.png" width="80%"></img>

## Usage

Set up `tableContents` in `viewDidLoad`:

```swift
import QuickTableViewController

final class ViewController: QuickTableViewController {

  override func viewDidLoad() {
    super.viewDidLoad()

    tableContents = [
      Section(title: "Switch", rows: [
        SwitchRow(text: "Setting 1", switchValue: true, action: { _ in }),
        SwitchRow(text: "Setting 2", switchValue: false, action: { _ in })
      ]),

      Section(title: "Tap Action", rows: [
        TapActionRow(text: "Tap action", action: { [weak self] in self?.showAlert($0) })
      ]),

      Section(title: "Navigation", rows: [
        NavigationRow(text: "CellStyle.default", detailText: .none, icon: .named("gear")),
        NavigationRow(text: "CellStyle", detailText: .subtitle(".subtitle"), icon: .named("globe")),
        NavigationRow(text: "CellStyle", detailText: .value1(".value1"), icon: .named("time"), action: { _ in }),
        NavigationRow(text: "CellStyle", detailText: .value2(".value2"))
      ], footer: "UITableViewCellStyle.Value2 hides the image view."),

      RadioSection(title: "Radio Buttons", options: [
        OptionRow(text: "Option 1", isSelected: true, action: didToggleSelection()),
        OptionRow(text: "Option 2", isSelected: false, action: didToggleSelection()),
        OptionRow(text: "Option 3", isSelected: false, action: didToggleSelection())
      ], footer: "See RadioSection for more details.")
    ]
  }

  // MARK: - Actions

  private func showAlert(_ sender: Row) {
    // ...
  }

  private func didToggleSelection() -> (Row) -> Void {
    return { [weak self] row in
      // ...
    }
  }

}
```

### NavigationRow

#### Detail Text Styles

```swift
NavigationRow(text: "UITableViewCellStyle.default", detailText: .none)
NavigationRow(text: "UITableViewCellStyle", detailText: .subtitle(".subtitle")
NavigationRow(text: "UITableViewCellStyle", detailText: .value1(".value1")
NavigationRow(text: "UITableViewCellStyle", detailText: .value2(".value2"))
```

[`Subtitle`](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/Model/Subtitle.swift) and the [initializers with title/subtitle](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/Model/Deprecated.swift) are deprecated and will be removed in **v2.0.0**.

#### Accessory Type

* The `NavigationRow` shows with different accessory types based on the `action` and `accessoryButtonAction` closures:

```swift
var accessoryType: UITableViewCell.AccessoryType {
  switch (action, accessoryButtonAction) {
  case (nil, nil):      return .none
  case (.some, nil):    return .disclosureIndicator
  case (nil, .some):    return .detailButton
  case (.some, .some):  return .detailDisclosureButton
  }
}
```

* The `action` will be invoked when the table view cell is selected.
* The `accessoryButtonAction` will be invoked when the accessory button is selected.

#### Images

```swift
enum Icon {
  case named(String)
  case image(UIImage)
  case images(normal: UIImage, highlighted: UIImage)
}
```

* Images in table view cells can be set by specifying the `icon` of each row.
* Table view cells in `UITableViewCellStyle.value2` will not show the image view.

### SwitchRow

* A `SwitchRow` is representing a table view cell with a `UISwitch` as its `accessoryView`.
* The `action` will be invoked when the switch value changes.

### TapActionRow

* A `TapActionRow` is representing a button-like table view cell.
* The `action` will be invoked when the table view cell is selected.
* The icon, detail text, and accessory type are disabled in `TapActionRow`.

### OptionRow

* An `OptionRow` is representing a table view cell with `.checkmark`.
* The `action` will be invoked when the selected state is toggled.

```swift
let didToggleSelection: (Row) -> Void = { [weak self] in
  if let option = $0 as? OptionRowCompatible, option.isSelected {
    // to exclude the event where the option is toggled off
  }
}
```

### RadioSection

* `RadioSection` allows only one selected option at a time.
* Setting `alwaysSelectsOneOption` to true will keep one of the options selected.
* `OptionRow` can also be used with `Section` for multiple selections.

## Customization

### Rows

All rows must conform to [`Row`](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/Protocol/Row.swift) and [`RowStyle`](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/Protocol/RowStyle.swift). Additional interface to work with specific types of rows are represented as different [protocols](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/Protocol/RowCompatible.swift):

* `NavigationRowCompatible`
* `OptionRowCompatible`
* `SwitchRowCompatible`
* `TapActionRowCompatible`

### Cell Classes

A customized table view cell type can be specified to rows during initialization.

```swift
// Default is UITableViewCell.
NavigationRow<CustomCell>(text: "Navigation", detailText: .none)

// Default is SwitchCell.
SwitchRow<CustomSwitchCell>(text: "Switch", switchValue: true, action: { _ in })

// Default is TapActionCell.
TapActionRow<CustomTapActionCell>(text: "Tap", action: { _ in })

// Default is UITableViewCell.
OptionRow<CustomOptionCell>(text: "Option", isSelected: true, action: { _ in })
```

Since the rows carry different cell types, they can be matched using either the concrete types or the related protocol:

```swift
let action: (Row) -> Void = {
  switch $0 {
  case let option as OptionRow<CustomOptionCell>:
    // only matches the option rows with a specific cell type
  case let option as OptionRowCompatible:
    // matches all option rows
  default:
    break
  }
}
```

### Overwrite Default Configuration

You can use `register(_:forCellReuseIdentifier:)` to specify custom cell types for the [table view](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/QuickTableViewController.swift#L100-L102) to use. See [CustomizationViewController](https://github.com/bcylin/QuickTableViewController/blob/develop/Example-iOS/ViewControllers/CustomizationViewController.swift) for the cell reuse identifiers of different rows.

Table view cell classes that conform to `Configurable` can take the customization during `tableView(_:cellForRowAt:)`:

```swift
protocol Configurable {
  func configure(with row: Row & RowStyle)
}
```

Additional setups can also be added to each row using the `customize` closure:

```swift
protocol RowStyle {
  var customize: ((UITableViewCell, Row & RowStyle) -> Void)? { get }
}
```

The `customize` closure [overwrites](https://github.com/bcylin/QuickTableViewController/blob/develop/Source/QuickTableViewController.swift#L104-L109) the `Configurable` setup.

### UIAppearance

As discussed in issue [#12](https://github.com/bcylin/QuickTableViewController/issues/12), UIAppearance customization works when the cell is dequeued from the storyboard. One way to work around this is to register nib objects to the table view. Check out [AppearanceViewController](https://github.com/bcylin/QuickTableViewController/blob/develop/Example-iOS/ViewControllers/AppearanceViewController.swift) for the setup.

## tvOS Differences

* `UISwitch` is replaced by a checkmark in `SwitchCell`.
* `TapActionCell` does not use center aligned text.
* `NavigationRow.accessoryButtonAction` is not available.
* Cell image view's left margin is 0.

## Limitation

> When to use **QuickTableViewController**?

QuickTableViewController is good for presenting static table contents, where the sections and rows don't change dynamically after `viewDidLoad`.

It's possible to update the table contents by replacing a specific section or row. Using different styles on each row requires additional configuration as described in the [Customization](#customization) section.

> When **not** to use it?

QuickTableViewController is not designed for inserting and deleting rows. It doesn't handle table view reload animation either. If your table view needs to update dynamically, you might want to consider other solutions such as [IGListKit](https://github.com/Instagram/IGListKit).

## Documentation

* [QuickTableViewController Reference](https://bcylin.github.io/QuickTableViewController)

## Installation

TODO: 

## License

QuickTableViewController is released under the MIT license.
See [LICENSE](https://github.com/bcylin/QuickTableViewController/blob/master/LICENSE) for more details.
Image source: [iconmonstr](http://iconmonstr.com/license/).
