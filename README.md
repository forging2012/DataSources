# DataSources

💾 🗄 Data-driven CollectionView Framework. (Can adapt ASCollectionNode)

---

> 🚨⚠️ WARNING ⚠️🚨<br>
This project is in a prerelease state. There is active work going on that will result in API changes that can/will break code while things are finished. Use with caution.

---

![](https://media.giphy.com/media/l1J3tmzS5WTBbTD5m/giphy.gif)

## Thanks

**Diff-algorithm**
- IGListKit-IGListDiff
  - https://github.com/Instagram/IGListKit
  - https://github.com/lxcid/ListDiff

## Usage (Example)

### Conform protocol `Diffable`

```swift
public protocol Diffable {
  associatedtype Identifier : Hashable
  var diffIdentifier: Identifier { get }
}
```

```swift
struct Model : Diffable {

  var diffIdentifier: String {
    return id
  }
  
  let id: String
}
```

### Single-Section (UICollectionView)

1. Define `SectionDataSource` in ViewController

```swift
let collectionView: UICollectionView
var models: [Model]

let sectionDataSource = SectionDataSource<Model, CollectionViewAdapter>(
  adapter: CollectionViewAdapter(collectionView: self.collectionView),
  displayingSection: 0,
  isEqual: { $0.id == $1.id } // If Model has Equatable, you can omit this closure.
)
```

2. Bind Models to `SectionDataSource` in ViewController

```swift
var models: [Model] = […] {
  didSet {
    sectionDataSource.update(items: items, updateMode: .partial(animated: true), completion: {
      // Completed update
    })
  }
}
```

3. Implement UICollectionViewDataSource in ViewController

```swift
func numberOfSections(in collectionView: UICollectionView) -> Int {
  return 1
}

func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
  return sectionDataSource.numberOfItems()
}

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {

  let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! Cell
  let m = sectionDataSource.item(for: indexPath)
  cell.label.text = m.title
  return cell
}
```

### Multiple-Section (UICollectionView)

1. Define `DataSource` in ViewController

```swift
let collectionView: UICollectionView
var models: [Model]

let dataSource = DataSource<CollectionViewAdapter>(adapter: CollectionViewAdapter(collectionView: self.collectionView))
```

2. Define `SectionContext<T>` in ViewController

```swift
let section0 = SectionContext.init(ModelA.self, section: 0, isEqual: { $0.id == $1.id })
let section1 = SectionContext.init(ModelB.self, section: 1, isEqual: { $0.id == $1.id })
```

3. Add `SectionContext` to `DataSource`

```swift
dataSource.addSectionDataSource(context: section0)
dataSource.addSectionDataSource(context: section1)
```

4. Bind Models to DataSource

```swift
var section0Models: [ModelA] = […] {
  didSet {
    dataSource.update(
      in: section0,
      items: section0Models,
      updateMode: .partial(animated: true),
      completion: {
        
    })
  }
}

var section1Models: [ModelA] = […] {
  didSet {
    dataSource.update(
      in: section0,
      items: section1Models,
      updateMode: .partial(animated: true),
      completion: {
        
    })
  }
}
```

5. Implement UICollectionViewDataSource

```swift
func numberOfSections(in collectionView: UICollectionView) -> Int {
  return dataSource.numberOfSections()
}

func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
  return dataSource.numberOfItems(in: section)
}

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {

  return dataSource.item(
    at: indexPath,
    returnType: UICollectionViewCell.self,
    handlers: [
    .init(context: section0) { (m: ModelA) in
      let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! Cell
      cell.label.text = m.title
      return cell
    },
    .init(context: section1) { (m: ModelB) in
      let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! Cell
      cell.label.text = m.title
      return cell
      },
    ])

  /* Other way
  switch indexPath.section {
  case section0:
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! Cell
    let m = _dataSource.item(at: indexPath, in: section0)
    cell.label.text = m.title
    return cell
  case section1:
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "Cell", for: indexPath) as! Cell
    let m = _dataSource.item(at: indexPath, in: section1)
    cell.label.text = m.title
    return cell
  default:
    fatalError()
  }
   */
}
```

# Author

muukii, m@muukii.me

# License

**DataSources** is available under the MIT license. See the LICENSE file for more info.
