
## Proposal to decouple BVC from Keyboard shortcuts handling

Hi everyone,

I would like to bring up a topic for discussion here in the group and also in our upcoming iOS engineering meetings. Given that the BVC is quite a large view controller, it would be beneficial to decouple it from some of its responsibilities.

In conversations with Winnie, we came to the conclusion that the UIKeyCommands registered inside the BVC could be encapsulated in a separate class. This would be helpful since most of these commands either already work or will eventually be implemented with Redux.

There is, however, an important constraint with the solution I’m proposing, and I’d like to outline why this approach is necessary. Essentially, for the key commands’ actions to be triggered, they must exist within the BVC class or one of its subviews, as they follow the chain-of-responsibility pattern. This means that the command is dispatched through the chain of controllers and all UIResponder subclasses until it finds one that can handle it. Because of this, it’s not possible to have a generic class that both implements all the commands and builds the list of them independently.

With this in mind, the solution I propose is to create a simple BVC controller decorator that wraps the BVC and introduces the keyboard shortcuts functionality.

```swift
final class BrowserViewControllerShortcutsDecorator: UIViewController {
    // Should be accessible since other part of code may need to use it
    let bvc: BrowserViewController

    init(bvc: BrowserViewController) {
        self.bvc = bvc
    }

    func viewDidLoad() {
        // add bvc to children of decorator 
        // and constraint to edges of decorator    
    }

    // MARK: - KeyCommands

    override var keyCommands: [UIKeyCommand]? {
        // build all the list of commands
    }

    // Those methods coul'd be internal since the UIMenuBuilder needs to select those
    // otherwise that is not built inside this class

    @objc
    private func reloadWebPage() {
        store.dispatch(GeneralBrowserAction(windowUUID: windowUUID, 
                                            actionType: GeneralBrowserActionType.reloadPage))
    }

    @objc
    private func nextPage() {
        store.dispatch(...)
    } 

    @objc
    private func previousPage() {
        store.dispatch(...)
    }
}

class BrowserCoordinator: ... {
    var browserViewController: BrowserViewController {
        return browserShortcutsDecorator.bvc
    }
    private var browserShortcutsDecorator: BrowserViewControllerShortcutsDecorator

    init(...) {
        let bvc = BrowserViewController(...)
        self.browserShortcutsDecorator = BrowserViewControllerShortcutsDecorator(bvc: bvc)
    }
}

```