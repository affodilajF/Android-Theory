# NAVIGATION 

LiFo => Last in first out. 

### Backstack
Stack of navigation destinations that the user has navigated through within the app. 

### Clearing Backstack 
Use popUpTo(0) with ZERO PARAMETER. Means the all stack except screen A will be cleared. 
![image](https://github.com/user-attachments/assets/66af2e7d-4770-445a-9f78-919e711a75c2)

# UTILITY 

### Navigate.popBackStack(@IdRes destinationId: Int, inclusive: Boolean)
- all stacks above destinationId will be poped. 
- destinationId => rute tujuan
- inclusive =>
  - true => rute tujuan ikut dihapus (sampai on destroy) 
  - false => rute tujuan tidak ikut dihapus

## Navigate(){ popUpTo("destinationScreen")}
- all stack dibawah destinationScreen akan dihapus
