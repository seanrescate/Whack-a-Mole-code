#include <Servo.h>
#include <LiquidCrystal.h>
#include <Keypad.h>

Servo myservo;
LiquidCrystal lcd(50, 2, 3, 53, 4, 52);

const int trigPin = 10;
const int echoPin = 11;

const byte ROWS = 4;
const byte COLS = 4;

char hexaKeys[ROWS][COLS] =
{
    {'1', '2', '3', 'A'},
    {'4', '5', '6', 'B'},
    {'7', '8', '9', 'C'},
    {'*', '0', '#', 'D'}
};

byte rowPins[ROWS] = {5, 40, 41, 6};
byte colPins[COLS] = {7, 51, 8, 9};

Keypad customKeypad = Keypad(
    makeKeymap(hexaKeys),
    rowPins,
    colPins,
    ROWS,
    COLS
);

const int PLAYER_SIZE = 4;

int ledState1 = LOW;
int ledState2 = LOW;
int ledState3 = LOW;
int ledState4 = LOW;

int secondz = 0;
int counter = 0;

const int INTERVAL = 1000;
const int min = 1;
const int max = 5;

float duration, distance;

static int lightState = 0;
int seconds = 0;

unsigned long startingTime1 = 0; // mole 1
unsigned long startingTime2 = 0; // mole 2
unsigned long startingTime3 = 0; // mole 3
unsigned long startingTime4 = 0; // mole 4
unsigned long startingTime5 = 0; // lcd display
unsigned long startingTime6 = 0; // ping sensor
unsigned long startingTime7 = 0; // servo motor

void GameState(int);
void MenuState();
void mole1(); // just wanted the lights to go off randomly, so I called void
void mole2();
void mole3();
void mole4();

static int mode = 0;
static int nextLife = 0;

static int playerScores[PLAYER_SIZE] = {0, 0, 0, 0};
static int currentPlayer = 0;


void setup()
{
    // put your setup code here, to run once:

    myservo.attach(12);

    pinMode(42, OUTPUT); // life 1
    pinMode(44, OUTPUT); // life 2
    pinMode(46, OUTPUT); // life 3

    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);

    // mole 1
    pinMode(22, OUTPUT);
    pinMode(23, INPUT);
    pinMode(25, INPUT);

    // mole 2
    pinMode(26, OUTPUT);
    pinMode(27, INPUT);
    pinMode(29, INPUT);

    // mole 3
    pinMode(30, OUTPUT);
    pinMode(31, INPUT);
    pinMode(33, INPUT);

    // mole 4
    pinMode(34, OUTPUT);
    pinMode(35, INPUT);
    pinMode(37, INPUT);

    lcd.begin(16, 2);
}


void loop()
{
    switch (mode)
    {
        case 0:
            MenuState();
            break;

        case 1:
            GameState(1);
            LifeState(nextLife);
            break;

        case 2:
            GameState(2);
            LifeState(nextLife);
            break;

        case 3:
            GameState(3);
            LifeState(nextLife);
            break;

        case 4:
            GameState(4);
            LifeState(nextLife);
            break;
    }

    digitalWrite(trigPin, LOW);
    delayMicroseconds(5);

    digitalWrite(trigPin, HIGH);
    delayMicroseconds(10);

    digitalWrite(trigPin, LOW);

    duration = pulseIn(echoPin, HIGH);
    distance = (duration * .0343) / 2;

    delay(100);

    if (distance < 15)
    {
        if (millis() - startingTime6 >= 750)
        {
            startingTime6 = millis();

            delay(250);
            LifeState(nextLife);
            nextLife++;
        }
    }
    else if (distance > 15)
    {
        startingTime6 = millis();
    }

    if (millis() - startingTime7 > 3000)
    {
        startingTime7 = millis();
        ServoMovement();
    }
}


void LifeState(int nextLife)
{
    switch (nextLife)
    {
        case 0:
            digitalWrite(42, HIGH);
            digitalWrite(44, HIGH);
            digitalWrite(46, HIGH);
            break;

        case 1:
            digitalWrite(42, LOW);
            digitalWrite(44, HIGH);
            digitalWrite(46, HIGH);
            break;

        case 2:
            digitalWrite(42, LOW);
            digitalWrite(44, LOW);
            digitalWrite(46, HIGH);
            break;

        case 3:
            digitalWrite(42, LOW);
            digitalWrite(44, LOW);
            digitalWrite(46, LOW);
            break;
    }
}


long microsecondsToInches(long microseconds)
{
    return microseconds / 74 / 2;
}


long microsecondsToCentimeters(long microseconds)
{
    return microseconds / 29 / 2;
}


void ServoMovement() // come back to change angle of rotation
{
    int movementOfServo[4] = {100, 125, 150, 175};

    static int position = 0;

    startingTime7 = millis();

    myservo.write(movementOfServo[position]);

    position++;

    if (position == 4)
        position = 0;
}


void MenuState()
{
    lcd.setCursor(0, 0);
    lcd.print("players 1-4");

    lcd.setCursor(0, 1);

    char customKey = customKeypad.getKey();

    switch (customKey)
    {
        case '1':
            lcd.print("valid");
            delay(1000);
            lcd.clear();
            mode = 1;
            break;

        case '2':
            lcd.print("valid");
            delay(1000);
            lcd.clear();
            mode = 2;
            break;

        case '3':
            lcd.print("valid");
            delay(1000);
            lcd.clear();
            mode = 3;
            break;

        case '4':
            lcd.print("valid");
            delay(1000);
            lcd.clear();
            mode = 4;
            break;

        case 'A':
        case 'B':
        case 'C':
        case 'D':
        case '5':
        case '6':
        case '7':
        case '8':
        case '9':
        case '*':
        case '0':
        case '#':
            lcd.begin(16, 2);
            lcd.print("players 1-4");
    }
}


void GameState(int playerSize)
{
    static int playerScores[PLAYER_SIZE] = {0, 0, 0, 0};
    static int currentPlayer = 0;

    mole1();
    mole2();
    mole3();
    mole4();

    lcd.setCursor(0, 0);
    lcd.print("Plyr ");
    lcd.print(currentPlayer + 1);
    lcd.print("score: ");

    if (millis() - startingTime5 >= INTERVAL && secondz <= 60)
    {
        // a timer for the lcd

        startingTime5 = millis();

        lcd.setCursor(0, 1);
        lcd.print(secondz + 1);

        secondz++;
    }

    if ((digitalRead(23) == HIGH && digitalRead(25) == HIGH)) // mole 1
    {
        digitalWrite(22, LOW);
        delay(250);

        counter++;

        lcd.setCursor(14, 0);
        lcd.print(playerScores[currentPlayer] + 1);

        // adding the score per player
        playerScores[currentPlayer]++;
    }

    if ((digitalRead(27) == HIGH && digitalRead(29) == HIGH)) // mole 2
    {
        digitalWrite(26, LOW);
        delay(250);

        counter++;

        lcd.setCursor(14, 0);
        lcd.print(playerScores[currentPlayer] + 1);

        playerScores[currentPlayer]++;
    }

    if ((digitalRead(31) == HIGH && digitalRead(33) == HIGH)) // mole 3
    {
        digitalWrite(30, LOW);
        delay(250);

        counter++;

        lcd.setCursor(14, 0);
        lcd.print(playerScores[currentPlayer] + 1);

        playerScores[currentPlayer]++;
    }

    if ((digitalRead(35) == HIGH && digitalRead(37) == HIGH)) // mole 4
    {
        digitalWrite(34, LOW);
        delay(250);

        counter++;

        lcd.setCursor(14, 0);
        lcd.print(playerScores[currentPlayer] + 1);

        playerScores[currentPlayer]++;
    }

    if (secondz == 61 || nextLife == 3)
    {
        LifeState(3);

        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print('P');
        lcd.print(currentPlayer + 1);

        lcd.setCursor(0, 1);
        lcd.print("total: ");
        lcd.print(playerScores[currentPlayer]);

        digitalWrite(22, LOW);
        digitalWrite(26, LOW);
        digitalWrite(30, LOW);
        digitalWrite(34, LOW);

        delay(5000);

        lcd.clear();
        lcd.setCursor(1, 1);
        lcd.print(' ');

        currentPlayer++;

        nextLife = 0;
        secondz = 0;
    }

    if (currentPlayer + 1 > playerSize)
    {
        lcd.setCursor(0, 0);
        lcd.print("P1: ");
        lcd.print(playerScores[0]);

        lcd.setCursor(8, 0);
        lcd.print("P2: ");
        lcd.print(playerScores[1]);

        lcd.setCursor(0, 1);
        lcd.print("P3: ");
        lcd.print(playerScores[2]);

        lcd.setCursor(8, 1);
        lcd.print("P4: ");
        lcd.print(playerScores[3]);

        for (int i = 0; i < PLAYER_SIZE; i++)
        {
            playerScores[i] = 0;
            currentPlayer = 0;
        }

        delay(5000);
        lcd.clear();

        mode = 0;
    }
}


void mole1()
{
    if (millis() - startingTime1 >= INTERVAL)
    {
        startingTime1 = millis();

        if (random(min, max) == 3)
            ledState1 = HIGH;
        else
            ledState1 = LOW;
    }

    digitalWrite(22, ledState1);
}


void mole2()
{
    if (millis() - startingTime2 >= INTERVAL)
    {
        startingTime2 = millis();

        if (random(min, max) == 3)
            ledState2 = HIGH;
        else
            ledState2 = LOW;
    }

    digitalWrite(26, ledState2);
}


void mole3()
{
    if (millis() - startingTime3 >= INTERVAL)
    {
        startingTime3 = millis();

        if (random(min, max) == 3)
            ledState3 = HIGH;
        else
            ledState3 = LOW;
    }

    digitalWrite(30, ledState3);
}


void mole4()
{
    if (millis() - startingTime4 >= INTERVAL)
    {
        startingTime4 = millis();

        if (random(min, max) == 3)
            ledState4 = HIGH;
        else
            ledState4 = LOW;
    }

    digitalWrite(34, ledState4);
}
