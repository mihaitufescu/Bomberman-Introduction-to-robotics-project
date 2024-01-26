# Bomberman-Introduction-to-robotics-project
# Matrix mini-game - Bomberman

## Requirement:
Develop a small game on the 8x8 matrix.
- The game must have at least 3 types of elements: player (blinks slowly), bombs/bullets (blinks fast), wall (doesn’t blink). The game chosen in this case is Bomberman.
- Create a menu for your game. You should scroll on the LCD with the joystick. 
- While playing the game display all relevant info.
Please check the detailed requirement here.

## Step 1: Test each physical componenent's functionality
For this step you will need:
- Arduino Uno Board
- 8x8 Matrix
- MAX7219 Driver
- Resistors as needed
- Breadboard
- Joystick
- Buzzer
- Jump wires
- LCD Display

Test the components:

- Connect the Matrix:
    - Connect the 8x8 matrix to the MAX7219 driver.
    - Connect the MAX7219 driver to the Arduino Uno.
    - Ensure proper voltage levels using resistors if needed.

- Joystick Connection:
    - Connect the analog joystick to the Arduino Uno.
    - Connect the X and Y pins of the joystick to A0 and A1 pins, respectively.
    - Connect the switch pin of the joystick to A2 pin.

- Buzzer Connection:
    - Connect the buzzer to the Arduino Uno.
    - Ensure proper resistance if needed.

- LCD Display Connection:
    - Connect the LCD display to the Arduino Uno.
    - Utilize the joystick to scroll on the LCD display for menu navigation.

- Overall Circuit:
    - Connect all components on the breadboard, ensuring proper power and ground connections.
    - Use jump wires to establish connections between components.

Circuit implementation:
<img src="https://github.com/mihaitufescu/Bomberman-Introduction-to-robotics-project/blob/main/Resources/Images/20231130_222836.jpg" alt="Circuit implementation" width="50%" height="50%" />

Example code:

```
// Constants
const int dPin = 12;
const int cPin = 7;
const int lPin = 10;
const byte matSize = 8;

// Controller
const int xPin = A0;
const int yPin = A1;
const int pinSW = A2;

// LedControl instance
LedControl lc = LedControl(dPin, cPin, lPin, 1);

// Game settings
byte gameMapBrightness = 2;
byte gameMap[8][8];

// Player position
byte playerPosX = 0;
byte playerPosY = 0;
byte lastPlayerPosX = 0;
byte lastPlayerPosY = 0;

//Joystick variables used to set its sensitivity
const int minThreshold = 200;
const int maxThreshold = 600;
const byte moveInterval = 100;
unsigned long long lastMoved = 0;
bool mapHasChanged = true;

// Button and Bomb state
boolean buttonState = false;
boolean bombIsPlaced = false;
int bombPosX = -1;
int bombPosY = -1;
int explosionTime = 2000;
int lastPositionSetTime;
int currLevel = 1;

// LCD display pins
unsigned int lcdBrightness = 255;
const byte rsPin = 9;
const byte ePin = 8;
const byte d4Pin = 11;
const byte d5Pin = 2;
const byte d6Pin = 5;
const byte d7Pin = 4;
const byte lcdBacklight = 6;
LiquidCrystal lcdScreen(rsPin, ePin, d4Pin, d5Pin, d6Pin, d7Pin);

//Menu variables
unsigned int menu = 1;
unsigned int subMenu = 1;
bool gameOn = false;
bool settingsOn = false;
bool adjustingDifficulty = false;
bool adjustingLCDBrightness = false;
bool adjustingMatrixBrightness = false;
bool highScoreOn = false;
bool htpOn = false;
unsigned int difficultyOption  = 1;
unsigned int htpMenu = 1;
unsigned int highscoresMenu = 1;

const int debounceDelay = 200;
unsigned long lastDebounceTime = 0;
unsigned long lastDebounceTimeX = 0;
unsigned long lastDebounceTimeY = 0;
unsigned int lifes = 3; 
unsigned long lastScrollTime = 0;
unsigned long scrollInterval = 500;

const int buzzerPin = 3;
const int menuFreq = 500;
const int bombExplodeSound = 1000;
unsigned long lastBuzzerSoundTime = 0;
unsigned int buzzerDuration = 300;
struct HighScores {
  int score1;
  int score2;
  int score3;
};
int highScore1 = 0;
int highScore2 = 0;
int highScore3 = 0;
const int highScoreStartAddr = 30;  
```
## Step 2: Structure of the code and data initialization

This code is structured into several methods and data structures for avoiding code redudancy and performance:
- Handling main functionalities into separate methods:

    1. generateRandomgameMap: This function is responsible for generating a random game map with a guaranteed path from the top-left to the bottom-right. It places walls on the map while ensuring a valid path.

    2. setgameMap: Initializes the LED matrix display and sets up the initial state of the game map. It involves configuring the display and generating the initial random game map.

    3. resetgameMap: Resets the game map to its initial state or generates a new random game map. It turns off every LED in the matrix and sets the player's position.

    4. mapUpdate: Updates the LED matrix display based on the current state of the game map. It iterates through the game map and updates the LEDs accordingly.

    5. playerPosUpdate: Updates the player's position based on analog joystick readings. It reads the joystick values and moves the player within the valid bounds of the game map.

    6. clearCell: Clears the specified cell in the game map and updates the LED matrix display.

    7. clearAdjacentCells: Clears adjacent cells around the specified position in the game map.

    8. isValidPosition: Checks if the specified position is within the valid bounds of the game map.

    9. isPlayerInBombRange: Checks if the player is in the bomb range based on their positions.

    10. handlePlayerBlinking: Handles the blinking of the player on the LED matrix at a regular interval.

    11. handleRapidBlinking: Handles the rapid blinking of the bomb on the LED matrix at a faster interval.

    12. updateMenu: Updates the LCD menu display based on the current menu state.

    13. gameSettings: Functionality related to handling game settings.

    14. writeHighScores: Writes high scores to EEPROM (Electrically Erasable Programmable Read-Only Memory).

    15. readHighScores: Reads high scores from EEPROM.

## Step 3: Finalizing the code

The code is an Arduino sketch for a maze-like game with an LED matrix display. Players navigate a maze, place bombs, and aim for high scores. The sketch includes functions for game mechanics, menu navigation, settings adjustment, and high score management.

Code:

```
#include "LedControl.h"
#include "LiquidCrystal.h"
#include <EEPROM.h>
// Constants
const int dPin = 12;
const int cPin = 7;
const int lPin = 10;
const byte matSize = 8;

// Controller
const int xPin = A0;
const int yPin = A1;
const int pinSW = A2;

// LedControl instance
LedControl lc = LedControl(dPin, cPin, lPin, 1);

// Game settings
byte gameMapBrightness = 2;
byte gameMap[8][8];

// Player position
byte playerPosX = 0;
byte playerPosY = 0;
byte lastPlayerPosX = 0;
byte lastPlayerPosY = 0;

//Joystick variables used to set its sensitivity
const int minThreshold = 200;
const int maxThreshold = 600;
const byte moveInterval = 100;
unsigned long long lastMoved = 0;
bool mapHasChanged = true;

// Button and Bomb state
boolean buttonState = false;
boolean bombIsPlaced = false;
int bombPosX = -1;
int bombPosY = -1;
int explosionTime = 2000;
int lastPositionSetTime;
int currLevel = 1;

// LCD display pins
unsigned int lcdBrightness = 255;
const byte rsPin = 9;
const byte ePin = 8;
const byte d4Pin = 11;
const byte d5Pin = 2;
const byte d6Pin = 5;
const byte d7Pin = 4;
const byte lcdBacklight = 6;
LiquidCrystal lcdScreen(rsPin, ePin, d4Pin, d5Pin, d6Pin, d7Pin);

//Menu variables
unsigned int menu = 1;
unsigned int subMenu = 1;
bool gameOn = false;
bool settingsOn = false;
bool adjustingDifficulty = false;
bool adjustingLCDBrightness = false;
bool adjustingMatrixBrightness = false;
bool highScoreOn = false;
bool htpOn = false;
unsigned int difficultyOption  = 1;
unsigned int htpMenu = 1;
unsigned int highscoresMenu = 1;

const int debounceDelay = 200;
unsigned long lastDebounceTime = 0;
unsigned long lastDebounceTimeX = 0;
unsigned long lastDebounceTimeY = 0;
unsigned int lifes = 3; 
unsigned long lastScrollTime = 0;
unsigned long scrollInterval = 500;

const int buzzerPin = 3;
const int menuFreq = 500;
const int bombExplodeSound = 1000;
unsigned long lastBuzzerSoundTime = 0;
unsigned int buzzerDuration = 300;
struct HighScores {
  int score1;
  int score2;
  int score3;
};
int highScore1 = 0;
int highScore2 = 0;
int highScore3 = 0;
const int highScoreStartAddr = 30;  
// Function prototypes
void generateRandomgameMap();
void setgameMap();
void resetgameMap();
void mapUpdate();
void playerPosUpdate();
void clearCell(int x, int y);
void clearAdjacentCells(int x, int y);
bool isValidPosition(int x, int y);
bool isPlayerInBombRange(int playerX, int playerY, int bombX, int bombY);
void handlePlayerBlinking(byte x, byte y);
void handleRapidBlinking(byte x, byte y);
void updateMenu();
void gameSettings();
void writeHighScores(const HighScores& scores);
HighScores readHighScores();

void setup() {
  Serial.begin(9600);
  pinMode(pinSW, INPUT_PULLUP);
  pinMode(buzzerPin, OUTPUT);
  pinMode(lcdBacklight, OUTPUT);
  HighScores storedScores = readHighScores();
  highScore1 = storedScores.score1;
  highScore2 = storedScores.score2;
  highScore3 = storedScores.score3;
  analogWrite(lcdBacklight, 128);
  lcdScreen.begin(16, 2);
  updateMenu();
  randomSeed(analogRead(A3));
  setgameMap();
  gameMap[playerPosX][playerPosY] = 1;
}

void loop() {
  boolean currBtnState = digitalRead(pinSW);
  // Toggle button state when button is pressed with debounce
  if (currBtnState == LOW && (millis() - lastDebounceTime) > debounceDelay) {
    buttonState = !buttonState;
    lastDebounceTime = millis();
  }

  if (gameOn == false) {
    int yValue = analogRead(xPin);
    if(millis()-lastBuzzerSoundTime > buzzerDuration){
      noTone(buzzerPin);
      lastBuzzerSoundTime = millis();
    }
    if (settingsOn) {
      if(adjustingLCDBrightness){
        if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          if(lcdBrightness<255)
            lcdBrightness++;
          lastDebounceTimeY = millis();
          handleLCDBrightness();
        } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          if(lcdBrightness>50)
            lcdBrightness--;
          lastDebounceTimeY = millis();
          handleLCDBrightness();
        }
      }
      else if(adjustingMatrixBrightness){
        if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          if(gameMapBrightness<9)
            gameMapBrightness--;
          lastDebounceTimeY = millis();
          handleMatrixBrightness();
        } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          if(gameMapBrightness>0)
            gameMapBrightness++;
          lastDebounceTimeY = millis();
          handleMatrixBrightness();
        }
      }
      else if(adjustingDifficulty){
        if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          difficultyOption--;
          lastDebounceTimeY = millis();
          difficultyHandler();
        } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          difficultyOption++;
          lastDebounceTimeY = millis();
          difficultyHandler();
        }
      }
      else{
        if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          subMenu--;
          lastDebounceTimeY = millis();
          updateSettings();
        } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          subMenu++;
          lastDebounceTimeY = millis();
          updateSettings();
        }
      }
      if (buttonState == true && (millis() - lastDebounceTime > debounceDelay)) {
        if(adjustingDifficulty){
          adjustingDifficulty = false;
          updateSettings();
        }
        else if(adjustingMatrixBrightness){
          adjustingMatrixBrightness = false;
          lc.setIntensity(0, gameMapBrightness);
          updateSettings();
        }
        else if(adjustingLCDBrightness){
          adjustingLCDBrightness = false;
          analogWrite(lcdBacklight, lcdBrightness);
          updateSettings();
        }
        else{
          executeSettings();
        } 
        buttonState = false;
        lastDebounceTime = millis();
      }
    }
    else if(htpOn){
        if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          htpMenu--;
          lastDebounceTimeY = millis();
        } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          htpMenu++;
          lastDebounceTimeY = millis();
        }
      }
    else if(highScoreOn){
      if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
        tone(buzzerPin, menuFreq);
        highscoresMenu--;
        lastDebounceTimeY = millis();
        highScoreDisplay();
      } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
          tone(buzzerPin, menuFreq);
          highscoresMenu++;
          lastDebounceTimeY = millis();
          highScoreDisplay();
      }
      if(buttonState == true && (millis() - lastDebounceTime > debounceDelay)){
        highScoreOn = false;
        updateMenu();
        delay(400);
      }
    }
    else {
      if (yValue < minThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
        tone(buzzerPin, menuFreq);
        menu--;
        lastDebounceTimeY = millis();
        updateMenu();
      } else if (yValue > maxThreshold && (millis() - lastDebounceTimeY) > debounceDelay) {
        tone(buzzerPin, menuFreq);
        menu++;
        lastDebounceTimeY = millis();
        updateMenu();
      }
      if (buttonState == true && (millis() - lastDebounceTime > debounceDelay)) {
        executeAction();
        buttonState = false;
        lastDebounceTime = millis();
      }
    }
  } else {
    if(millis()-lastBuzzerSoundTime > buzzerDuration){
      noTone(buzzerPin);
      lastBuzzerSoundTime = millis();
    }
    // Update player position at a regular interval
    if (millis() - lastMoved >= moveInterval) {
      lcdScreen.setCursor(0,0);
      lcdScreen.print("Lifes:");
      lcdScreen.setCursor(0,1);
      lcdScreen.print(lifes);
      lastPlayerPosX = playerPosX;
      lastPlayerPosY = playerPosY;
      playerPosUpdate();

      // Place bomb if button is pressed and there is no bomb placed already
      if (buttonState == true && !bombIsPlaced && (millis() - lastDebounceTime > debounceDelay)) {
        tone(buzzerPin, bombExplodeSound);
        gameMap[lastPlayerPosX][lastPlayerPosY] = 1;
        lc.setLed(1, lastPlayerPosX, lastPlayerPosY, gameMap[lastPlayerPosX][lastPlayerPosY]);

        lastPositionSetTime = millis();

        bombIsPlaced = true;
        bombPosX = lastPlayerPosX;
        bombPosY = lastPlayerPosY;
        buttonState = false;
        lastDebounceTime = millis();
      }
      lastMoved = millis();
    }
    if (areNoWallsOnMap() == true){
      currLevel++;
      bombIsPlaced = false;
      resetgameMap();
    }
    // Handle rapid blinking if bomb is placed
    if (bombIsPlaced) {
      handleRapidBlinking(bombPosX, bombPosY);
    }

    // Handle bomb explosion after a certain time
    if (bombIsPlaced) {
    if (millis() - lastPositionSetTime >= explosionTime) {
      bombIsPlaced = false;

      // Check if player is in bomb range and reset gameMap if true
      if (isPlayerInBombRange(playerPosX, playerPosY, bombPosX, bombPosY)) {
        if (lifes == 1) {
          resetgameMap();
          lcdScreen.clear();
          lcdScreen.setCursor(0, 0);
          lcdScreen.print("Lost at level:");
          lcdScreen.setCursor(0, 1);
          lcdScreen.print(currLevel);
          delay(2000);
          gameOn = false;
        } else {
          delay(300);
          lifes--;
        }
      }

      // Clear bomb and adjacent cells
      clearCell(bombPosX, bombPosY);
      clearAdjacentCells(bombPosX, bombPosY);

      bombPosX = -1;
      bombPosY = -1;
      mapUpdate();
      }
    }

    // Update LED matrix display
    if (mapHasChanged) {
      mapUpdate();
      mapHasChanged = false;
    }

    // Handle player blinking
    handlePlayerBlinking(playerPosX, playerPosY);
  }
}

void updateMenu() {
  switch (menu) {
    case 0:
      menu = 1;
      break;
    case 1:
      lcdScreen.clear();
      lcdScreen.print(">START");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" SETTINGS");
      break;
    case 2:
      lcdScreen.clear();
      lcdScreen.print(">SETTINGS");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" HOW TO PLAY");
      break;
    case 3:
      lcdScreen.clear();
      lcdScreen.print(">HOW TO PLAY");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" START");
      break;
    case 4:
      lcdScreen.clear();
      lcdScreen.print(">HIGHSCORES");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" START");
      break;
    case 5:
      menu = 1;
      break;
  }
}
void executeAction() {
  switch (menu) {
    case 1:
      delay(1000);
      gameOn = true;
      lcdScreen.clear();
      break;
    case 2:
      delay(1000);
      settingsOn = true;
      lcdScreen.clear();
      updateSettings();
      break;
    case 3:
      delay(1000);
      htpOn = true;
      lcdScreen.clear();
      howToPlay();
      break;
    case 4:
      delay(1000);
      highScoreOn = true;
      lcdScreen.clear();
      highScoreDisplay();
      break;
  }
}
void updateSettings() {
  switch (subMenu) {
    case 0:
      subMenu = 1;
      break;
    case 1:
      lcdScreen.clear();
      lcdScreen.print(">EXIT");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" DIFFICULTY");
      break;
    case 2:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print(">DIFFICULTY");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" MATRIX BRIGHTNESS");
      break;
    case 3:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print(">MATRIX BRIGHTNESS");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" LCD BRIGHTNESS");
      break;
    case 4:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print(">LCD BRIGHTNESS");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(" DIFFICULTY");
      break;
    case 5:
      subMenu = 1;
      break;
  }
}
void executeSettings(){
  switch (subMenu) {
    case 1:
      settingsOn = false;
      updateMenu();
      break;
    case 2:
      adjustingDifficulty = true;
      lcdScreen.clear();
      difficultyHandler();
      break;
    case 3:
      adjustingMatrixBrightness = true;
      lcdScreen.clear();
      handleMatrixBrightness();
      break;
    case 4:
      adjustingLCDBrightness = true;
      lcdScreen.clear();
      handleLCDBrightness();
      break;
  }
}
void difficultyHandler(){
  lcdScreen.clear();
  lcdScreen.setCursor(0, 0);
  lcdScreen.print(">CHOOSE");
  lcdScreen.setCursor(0, 1);
  switch(difficultyOption){
    case 0:
      difficultyOption = 1;
    case 1:
      lcdScreen.print(">EASY");
      explosionTime = 4000;
      lifes = 5;
      break;
    case 2:
      lcdScreen.print(">MEDIUM");
      explosionTime = 3000;
      lifes = 4;
      break;
    case 3:
      lcdScreen.print(">HARD");
      explosionTime = 2000;
      lifes = 3;
      break;
    case 4:
      difficultyOption = 1;
      break;
  }
}
void highScoreDisplay() {
  lcdScreen.clear();
  HighScores storedScores = readHighScores();
  switch (highscoresMenu) {
    case 0:
      highscoresMenu = 1;
      break;
    case 1:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print("HIGHSCORE 1");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(storedScores.score1);
      break;
    case 2:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print("HIGHSCORE 2");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(storedScores.score2);
      break;
    case 3:
      lcdScreen.clear();
      lcdScreen.setCursor(0, 0);
      lcdScreen.print("HIGHSCORE 3");
      lcdScreen.setCursor(0, 1);
      lcdScreen.print(storedScores.score3);
      break;
    case 4:
      highscoresMenu = 1;  // Reset to the first case
      break;
  }
}
void handleLCDBrightness(){
  lcdScreen.clear();
  lcdScreen.setCursor(0, 0);
  lcdScreen.print(">CHOOSE (50-255)");
  lcdScreen.setCursor(0, 1);
  lcdScreen.print(lcdBrightness);
}
void handleMatrixBrightness(){
  lcdScreen.clear();
  lcdScreen.setCursor(0, 0);
  lcdScreen.print(">CHOOSE (0-9)");
  lcdScreen.setCursor(0, 1);
  lcdScreen.print(gameMapBrightness);
}
void howToPlay(){
  if (htpMenu == 1) {
  lcdScreen.clear();
  const char* messages[] = {
    "The goal of this game",
    " is to destroy",
    "every wall on the map",
    " on each level.",
    "Once a level finishes,",
    " a new one will respawn.",
    "Use the arrows to move.",
    "You can't move through the walls.",
    "Press the button in order",
    "to place a bomb. It will destroy",
    "every neighbour block.",
    "If you are near the bomb",
    "during the explosion,",
    "you will lose a life.",
    "At 0 lifes you lose.",
    "Scroll to exit."
  };

  const int numMessages = sizeof(messages) / sizeof(messages[0]);
  for (int i = 0; i < numMessages; i++) {
    lcdScreen.clear();
    lcdScreen.setCursor(0, 0);
    lcdScreen.print(messages[i]);
    int messageLength = strlen(messages[i]);
    int dynamicDelay = (messageLength > 18) ? 450 : 250;
    delay(dynamicDelay);

    for (int j = 0; j < 18; ++j) {
      lcdScreen.scrollDisplayLeft();
      delay(250);
      }
    }
  } 
  else{
    htpOn = false;
    lcdScreen.clear();
    lcdScreen.print(">EXIT");
    updateSettings();
    }
}

// Generate a random game map with a guaranteed path
void generateRandomgameMap() {
  // Initialize gameMap with a guaranteed path from top-left to bottom-right
  gameMap[0][0] = 0;
  gameMap[matSize - 1][matSize - 1] = 0;

  // Define the number of walls based on the current level
  int numWalls;
  switch (currLevel) {
    case 1:
      numWalls = 8;
      break;
    case 2:
      numWalls = 10;
      break;
    case 3:
      numWalls = 12;
      break;
    case 4:
      numWalls = 16;
      break;
    default:
      numWalls = 20;  // Default to 8 walls for unknown levels
      break;
  }

  // Generate random walls with a guarantee of a path
  for (int i = 0; i < numWalls; i++) {
    int row, col;
    do {
      row = random(1, matSize - 1);
      col = random(1, matSize - 1);
    } while (gameMap[row][col] != 0); // Make sure the selected cell is empty

    gameMap[row][col] = 1; // Set the wall
  }

  // Ensure a path from the top-left to bottom-right corners
  for (int row = 1; row < matSize - 1; row++) {
    for (int col = 1; col < matSize - 1; col++) {
      if (gameMap[row][col] == 1) {
        // Ensure there is at least one adjacent open cell
        if (!(gameMap[row - 1][col] == 0 || gameMap[row + 1][col] == 0 || gameMap[row][col - 1] == 0 || gameMap[row][col + 1] == 0)) {
          gameMap[row][col] = 0;
        }
      }
    }
  }
}


// Set up the initial LED matrix display
void setgameMap() {
  lc.shutdown(0, false);
  lc.setIntensity(0, gameMapBrightness);
  lc.clearDisplay(0);
  gameMap[playerPosX][playerPosY] = 1;
  generateRandomgameMap();
  mapUpdate();
}
bool areNoWallsOnMap() {
  for (int row = 0; row < matSize; row++) {
    for (int col = 0; col < matSize; col++) {
      // Check if any cell other than the player's position has a wall (LED on)
      if (!(row == playerPosX && col == playerPosY) && gameMap[row][col] == 1) {
        return false; // There is a wall on the map
      }
    }
  }
  return true; // No walls found on the map
}
// Reset the gameMap to the initial state or generate a new random gameMap
void resetgameMap() {
  // Turn off every LED in the matrix
  for (int row = 0; row < matSize; row++) {
    for (int col = 0; col < matSize; col++) {
      gameMap[row][col] = 0;
    }
  }
  bombIsPlaced = false;
  // Set the player's position
  gameMap[playerPosX][playerPosY] = 1;
  randomSeed(analogRead(A3));
  setgameMap();
  if (currLevel > highScore1) {
    highScore3 = highScore2;
    highScore2 = highScore1;
    highScore1 = currLevel;
  } else if (currLevel > highScore2) {
    highScore3 = highScore2;
    highScore2 = currLevel;
  } else if (currLevel > highScore3) {
    highScore3 = currLevel;
  }

  // Save updated high scores to EEPROM
  HighScores updatedScores = {highScore1, highScore2, highScore3};
  writeHighScores(updatedScores);
  delay(2000); // Delay to show the reset gameMap before continuing the game
}

// Update LED matrix display based on the gameMap array
void mapUpdate() {
  for (int row = 0; row < matSize; row++) {
    for (int col = 0; col < matSize; col++) {
      lc.setLed(0, row, col, gameMap[row][col]);
    }
  }
}

// Update player position based on analog joystick readings
void playerPosUpdate() {
  int xValue = analogRead(xPin);
  int yValue = analogRead(yPin);
  int hasMoved = false;
  byte newplayerPosX = playerPosX;
  byte newplayerPosY = playerPosY;

  // Move player based on joystick readings
  if (xValue < minThreshold && newplayerPosX > 0 && hasMoved == false) {
    newplayerPosX--;
    hasMoved = true;
  }else if (xValue > maxThreshold && newplayerPosX < matSize - 1 && hasMoved == false) {
    newplayerPosX++;
    hasMoved = true;
  }

  if (yValue < minThreshold && newplayerPosY < matSize - 1 && hasMoved == false) {
    newplayerPosY++;
    hasMoved = true;
  }else if (yValue > maxThreshold && newplayerPosY > 0 && hasMoved == false) {
    newplayerPosY--;
    hasMoved = true;
  }

  // Update gameMap if the new position is a valid open cell
  if (gameMap[newplayerPosX][newplayerPosY] == 0) {
    mapHasChanged = true;
    gameMap[playerPosX][playerPosY] = 0;
    gameMap[newplayerPosX][newplayerPosY] = 1;
    lastPlayerPosX = playerPosX;
    lastPlayerPosY = playerPosY;
    playerPosX = newplayerPosX;
    playerPosY = newplayerPosY;
  }
}

// Clear the specified cell in the gameMap and update the LED matrix
void clearCell(int x, int y) {
  if (isValidPosition(x, y)) {
    gameMap[x][y] = 0;
    lc.setLed(1, x, y, gameMap[x][y]);
  }
}

// Clear adjacent cells around the specified position
void clearAdjacentCells(int x, int y) {
  clearCell(x - 1, y);
  clearCell(x + 1, y);
  clearCell(x, y - 1);
  clearCell(x, y + 1);
}

// Check if the specified position is within the valid bounds of the gameMap
bool isValidPosition(int x, int y) {
  return x >= 0 && x < 8 && y >= 0 && y < 8;
}

// Check if the player is in the bomb range based on their positions
bool isPlayerInBombRange(int playerX, int playerY, int bombX, int bombY) {
  return (abs(playerX - bombX) <= 1 && abs(playerY - bombY) <= 1);
}

// Handle player blinking at a regular interval
void handlePlayerBlinking(byte x, byte y) {
  static unsigned long lastBlinkTime = 0;
  static bool isOn = true;
  static unsigned blinkInterval = 400;
  if (millis() - lastBlinkTime >= blinkInterval) {
    isOn = !isOn;
    lc.setLed(0, x, y, isOn);
    lastBlinkTime = millis();
  }
}

// Function to write high scores to EEPROM
void writeHighScores(const HighScores& scores) {
  EEPROM.put(highScoreStartAddr, scores);
}

// Function to read high scores from EEPROM
HighScores readHighScores() {
  HighScores scores;
  EEPROM.get(highScoreStartAddr, scores);
  return scores;
}

// Handle rapid blinking of the bomb at a faster interval
void handleRapidBlinking(byte x, byte y) {
  static unsigned long lastBlinkTime = 0;
  static bool isOn = true;
  static unsigned blinkInterval = 200;
  if (millis() - lastBlinkTime >= blinkInterval) {
    isOn = !isOn;
    lc.setLed(0, x, y, isOn);
    lastBlinkTime = millis();
  }
}

```

Demo :
https://youtube.com/shorts/69EAys8AuJA
