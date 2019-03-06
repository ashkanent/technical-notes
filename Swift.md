# iOS App Development

## Layout Notes
- Having three buttons aligned and show properly in the center:
  - from library drag a button. (then copy/paste two times)
  - drag and select all 3 buttons. From the bottom of the editor: **Embed In** > **stack view**
  - under _Attributes Inspector_ > **Stack View**, set _Alignment_ to "Center" and set _Distribution_ to "Fill Equally" and set _Spacing_ to "Use Standard Value"
  - then control drag the stack view to the ViewController and check **Center Horizontally in Safe Area** and **Center Vertically in Safe Area**

## Multiple MVCs
- This would be an app with multiple views. Each view represents a MVC.
- There are three types of multiple MVCs:
  1. tab bar view
    - like 4 tabs in the bottom (alarm app in iOS)
  2. split view (settings in iOS for iPad)
  3. navigation view (settings in iOS)

Here is an example of navigating through different views using segues. This would be "navigation view (3rd option above)" We'll change concentration app so first we select a theme and then load the cards using that theme:
- In `Main.storyboard`, from library, choose a `ViewController` and drag it into the board.
- Create a new class to bind it to this new ViewController. (`cmnd+N` > Cocoa Touch Class). Now to do the binding, go to the board, select the new ViewController, under _Identity Inspector_ > _Custom Class_ > _Class_ select the new view controller class that you just created.
_ The arrow on the left side of the MVC shows that's the first one to show when app lunches. You can drag it and put it on the left side of the new MVC (if you want it to be the first to show)

Now that we have two MVCs, we have to wire them up. We'll put three buttons in _ThemeChooserViewController_ and then control drag from each of these buttons to the other ViewController and select "_Action Segue_ > _Show_":
- it will show three arrows between two MVCs. select each and under "Attributes Inspector": _Storyboard Segue_ > _Identifier_ type something like "Choose Theme" (so we can use it in code to identify the segue)
- Now we need a navigation controller. select the _ThemeChooserViewController_ and from menubar: **Editor** > **Embed In** > **Navigation Controller**. (this will create a new view controller and puts it in the beginning)

Now we have to adjust the code to work. For that, in _ThemeChooserViewController_ we have to have the _prepare()_ function. This will prepare the segue so when we transition from one MVC to the next one, variables are set the way we want it.

**Note**: Everytime we go back and forth through segues, it shows a new, fresh MVC. So it won't be in the state that you left it before.

### iPad Compatible
Now to make it look good on iPad, we want to use _Split View_. To do this, got to story board, zoom out a lot! and from library drag the **Split View Controller**. By default it comes with Master/detail views, select them and delete!
- Control-drag from Split View controller to navigation controller and choose: **Relationship Segue** > **Master View Controller**
- Then control-drag to _ConcentrationViewController_ and choose: **Relationship Segue** > **Detail view controller**
- Now the three segues that we have from _ThemeChooserViewController_ to _ConcentrationViewController_ are of type _Show_ which was for navigating view. Select them and under **Attributes Inspector** > **Storyboard Segue** > **Kind** select _Show Detail_ which is for split view.

Now this should work and look good on iPad. The good thing is that it will work just fine in iPhone as it detects the device and shows navigation view in iPhone and split view in iPad (and iPhone plus landscape mode).

Now let's add a tab bar controller. Drag one from library to the story board. Now put this controller behind all of the above and control-drag from it to the _SplitViewController_ and select: **Relationship Segue** > **View controllers**.

- Views that are linked to this tab bar, now have a tab at bottom that you can rename
- in main view controller you'll see all those tab names and you can change their order

As a practice, we want to make segues from code (not from story board). Delete the three segues between 3 buttons and _ConcentrationViewController_. To do that:
- drag from those three buttons in storyboard to  _ThemeChooserViewController_ (the code)
  - Choose action, you can use _any_ as type (you have to cast) and you can call it _themeChooser_
- in storyboard drag from _ThemeChooserViewController_ (yellow button on top of it) to the _ConcentrationViewController_ and select: **Manual Segue** > **Show Detail**
  - same as before from _Attributes Inspector_ call it _Choose Theme_.
- in your code for the action you created, you just use `performSegue()` and now it should work.
- in short, we created one manual segue from _ThemeChooserViewController_ to _ConcentrationViewController_ (not from the buttons) and then in the code we created the segue.
- One advantage of doing this in code instead of storyboard is that for example we can do conditional segue! (like if the game is not finished do not go to other themes!)

Now whenever we run the app in iPhone, it doesn't go to the master view and opens the game with default theme. In order to fix it (make it show the theme chooser), we can use `awakeFromNib()` method to mark the theme chooser as the delegate of _SplitViewController_ and implement a method that it uses to collapse detail on top of master (look at the code for its code sample). So we modify this method in a way that it doesn't collapse on starting the game.

## Animation

Different kinds of animation:
1. Animating UIView properties
  - changing things like the frame or transparency
2. Animating Controller transitions
3. Core animation
4. OpenGL and Metal
  - 3D
5. SpriteKit
  - 2.5D animation (overlapping images moving around over each other, etc.)
6. Dynamic Animation
  - physics based animation (like when you swipe up iPhone)
