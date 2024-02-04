import itertools
import random

class MastermindAI:
    def __init__(self, code_length=4):
        self.code_length = code_length
        self.possible_colors = "RGBbYW"
        self.all_possible_codes = [''.join(code) for code in itertools.product(self.possible_colors, repeat=self.code_length)]
        self.remaining_codes = self.all_possible_codes.copy()
        self.previous_guess = None
        self.heuristic_scores = {code: 0 for code in self.all_possible_codes}

    def make_guess(self):
        if not self.previous_guess or len(self.remaining_codes) > 1:
            guess = self.choose_smart_guess()
        else:
            guess = self.generate_minimax_guess()

        self.previous_guess = guess
        return guess

    def choose_smart_guess(self):
        best_guesses = self.get_best_guesses()
        guess = random.choice(best_guesses)
        return guess

    def get_best_guesses(self):
        max_score = max(self.heuristic_scores.values())
        return [code for code, score in self.heuristic_scores.items() if score == max_score]

    def generate_minimax_guess(self):
        best_guess = self.minimax(2, float('-inf'), float('inf'))  # Adjust the depth of the minimax search
        return best_guess[0]

    def minimax(self, depth, alpha, beta):
        best_score = float('-inf')
        best_guess = None

        for code in self.remaining_codes:
            score = self.minimax_score(code, depth - 1, alpha, beta)
            if score > best_score:
                best_score = score
                best_guess = code
            alpha = max(alpha, best_score)
            if beta <= alpha:
                break

        return best_guess, best_score

    def minimax_score(self, guess, depth, alpha, beta):
        if depth == 0:
            return self.calculate_total_score(guess)

        score = float('inf')

        for code in self.remaining_codes:
            self.remaining_codes.remove(code)
            score = min(score, self.minimax_score(code, depth - 1, alpha, beta))
            self.remaining_codes.append(code)

            beta = min(beta, score)
            if beta <= alpha:
                break

        return score

    def calculate_total_score(self, guess):
        total_score = 0
        for code in self.remaining_codes:
            total_score += self.evaluate_guess(guess, code)
        return total_score

    def evaluate_guess(self, guess, code):
        correct_positions = sum(g == c for g, c in zip(guess, code))
        correct_colors = sum(min(guess.count(color), code.count(color)) for color in set(guess))
        incorrect_positions = correct_colors - correct_positions
        return correct_positions, incorrect_positions

    def update_heuristic_scores(self, correct_positions, incorrect_positions):
        updated_remaining_codes = []
        for code in self.remaining_codes:
            score = self.evaluate_guess(self.previous_guess, code)
            if score == (correct_positions, incorrect_positions):
                self.heuristic_scores[code] += 1
                updated_remaining_codes.append(code)

        self.remaining_codes = updated_remaining_codes

def display_legend():
    print("Color Code Legend:")
    print("R - Red")
    print("G - Green")
    print("B - Blue")
    print("Y - Yellow")
    print("W - White")
    print("b - Black")

def get_user_input(prompt):
    while True:
        try:
            return int(input(prompt))
        except ValueError:
            print("Invalid input. Please enter a number.")

def play_mastermind():
    code_length = 4
    max_guesses = 10
    ai = MastermindAI(code_length)

    print("Welcome to Mastermind! Try to guess the secret code.")
    display_legend()
    print(f"The secret code consists of colors: {ai.possible_colors}")
    print(f"The code length is: {code_length}")

    secret_code = ''.join(random.choice(ai.possible_colors) for _ in range(code_length))

    for attempt in range(1, max_guesses + 1):
        guess = ai.make_guess()
        print(f"\nAttempt {attempt}: AI's guess - {guess}")

        if guess == secret_code:
            print("\nCongratulations! AI guessed the correct code!")
            break

        correct_positions = get_user_input("Enter the number of correct positions: ")
        incorrect_positions = get_user_input("Enter the number of correct colors in incorrect positions: ")

        while correct_positions + incorrect_positions > code_length:
            print("Invalid input. The total number of correct positions and colors cannot exceed the code length.")
            correct_positions = get_user_input("Enter the number of correct positions: ")
            incorrect_positions = get_user_input("Enter the number of correct colors in incorrect positions: ")

        ai.update_heuristic_scores(correct_positions, incorrect_positions)

    print(f"\nThe secret code was: {secret_code}")

if __name__ == "__main__":
    play_mastermind()
