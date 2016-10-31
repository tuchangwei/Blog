title: MVVM Philosophy
date: 2016-10-31 15:59:02

categories: MVVM

tags: MVVM
---





## [Philosophy](https://github.com/devxoul/RxTodo)

- View doesn't have control flow. View cannot modify the data. View only knows how to map the data.

    **Bad**

  ```swift
    viewModel.title
        .map { $0 + "!" } // Bad: View should not modify the data
        .bindTo(self.titleLabel)
  ```

    **Good**

  ```swift
    viewModel.title
        .bindTo(self.titleLabel)
  ```

- View doesn't know what ViewModel does. View can only communicate to ViewModel about what View did.

    **Bad**

  ```swift
    viewModel.login() // Bad: View should not know what ViewModel does (login)
  ```

    **Good**

  ```swift
    self.loginButton.rx_tap
        .bindTo(viewModel.loginButtonDidTap) // "Hey I clicked the login button"

    self.usernameInput.rx_controlEvent(.EditingDidEndOnExit)
        .bindTo(viewModel.usernameInputDidReturn) // "Hey I tapped the return on username input"
  ```

- Model is hidden by ViewModel. ViewModel only exposes the minimum data so that View can render.

    **Bad**

  ```swift
    struct ProductViewModel {
        let product: Driver<Product> // Bad: ViewModel should hide Model
    }
  ```

    **Good**

  ```swift
    struct ProductViewModel {
        let productName: Driver<String>
        let formattedPrice: Driver<String>
        let formattedOriginalPrice: Driver<String>
        let originalPriceHidden: Driver<Bool>
    }
  ```

{%asset_img mvvm.png %}