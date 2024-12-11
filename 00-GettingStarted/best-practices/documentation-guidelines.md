# documentation-guidelines.md

Hey there, documentation explorer! 👋

Documentation is like leaving a map for future travelers (including yourself!) through your code. Today, we're going to learn how to create documentation that people will actually want to read and use. I remember when I first started coding, I thought documentation was just extra work - but I quickly learned how valuable good docs can be when you're trying to understand code months after writing it.

### Understanding Documentation's Purpose

Documentation serves multiple purposes in your development journey:

1. It helps others understand your code
2. It helps future-you remember why you made certain decisions
3. It makes collaboration easier
4. It reduces the time needed to onboard new team members

### Project-Level Documentation

Let's start with the big picture - your project's README.md file. This is like the cover letter for your code:

````markdown
# Project Name

A clear, concise description of what your project does and why it exists.

## Getting Started

These instructions will help you get a copy of the project up and running on your local machine.

### Prerequisites

What things you need to install and how to install them:

```bash
npm install
# or
yarn install
```
````

### Installation

Step by step guide to getting a development environment running:

1. Clone the repository

   ```bash
   git clone https://github.com/username/project.git
   ```

2. Install dependencies

   ```bash
   cd project
   npm install
   ```

3. Set up environment variables

   ```bash
   cp .env.example .env
   # Edit .env with your values
   ```

4. Start the development server
   ```bash
   npm run dev
   ```

## Project Structure

```plaintext
src/
├── components/    # Reusable UI components
├── pages/         # Route components
├── utils/         # Helper functions
├── services/      # API calls and business logic
└── styles/        # Global styles and themes
```

## Available Scripts

- `npm run dev`: Start development server
- `npm run build`: Build for production
- `npm run test`: Run test suite

````

### Component Documentation

When documenting React components, think about what another developer would need to know to use your component effectively:

```javascript
/**
 * UserProfile displays user information and handles profile updates.
 *
 * @component
 * @example
 * ```jsx
 * const userData = {
 *   name: 'John Doe',
 *   email: 'john@example.com'
 * };
 *
 * return (
 *   <UserProfile
 *     user={userData}
 *     onUpdate={handleUpdate}
 *     isEditable={true}
 *   />
 * );
 * ```
 */
const UserProfile = ({ user, onUpdate, isEditable = false }) => {
  // Component implementation...
};

UserProfile.propTypes = {
  /** User object containing profile information */
  user: PropTypes.shape({
    /** User's full name */
    name: PropTypes.string.isRequired,
    /** User's email address */
    email: PropTypes.string.isRequired,
    /** Optional profile picture URL */
    avatar: PropTypes.string
  }).isRequired,
  /** Callback function called when profile is updated */
  onUpdate: PropTypes.func.isRequired,
  /** Whether the profile can be edited */
  isEditable: PropTypes.bool
};

export default UserProfile;
````

### Function Documentation

Functions should be documented to explain what they do, what parameters they accept, and what they return:

```javascript
/**
 * Formats a number as a currency string with the specified currency symbol.
 * Handles negative numbers and decimal places appropriately.
 *
 * @param {number} amount - The amount to format
 * @param {string} [currency='USD'] - The currency code to use
 * @param {string} [locale='en-US'] - The locale to use for formatting
 * @returns {string} The formatted currency string
 *
 * @example
 * formatCurrency(1234.56)       // Returns "$1,234.56"
 * formatCurrency(1234.56, 'EUR') // Returns "€1,234.56"
 * formatCurrency(-1234.56)      // Returns "-$1,234.56"
 *
 * @throws {Error} If amount is not a number
 */
const formatCurrency = (amount, currency = "USD", locale = "en-US") => {
  if (typeof amount !== "number") {
    throw new Error("Amount must be a number");
  }

  return new Intl.NumberFormat(locale, {
    style: "currency",
    currency: currency,
  }).format(amount);
};
```

### API Documentation

When documenting APIs, be clear about endpoints, methods, parameters, and responses:

```javascript
/**
 * User API endpoints
 * @module services/userApi
 */

/**
 * Retrieves a user's profile information.
 *
 * @async
 * @function getUserProfile
 * @param {string} userId - The ID of the user to retrieve
 * @returns {Promise<Object>} User profile data
 *
 * @example
 * Response format:
 * {
 *   id: string,
 *   name: string,
 *   email: string,
 *   created: Date,
 *   settings: {
 *     theme: string,
 *     notifications: boolean
 *   }
 * }
 *
 * @throws {ApiError} When user is not found (404) or unauthorized (401)
 */
const getUserProfile = async (userId) => {
  // Implementation...
};
```

### Code Comments

Comments in your code should explain the why, not the what:

```javascript
// Bad - explains what (obvious from code)
// Set user to active
user.setActive(true);

// Good - explains why
// Enable account after email verification
user.setActive(true);

// Good - explains complex logic
// Calculate discount based on user's tier and purchase history
// Silver tier: 5% off if spent > $100
// Gold tier: 10% off if spent > $200
const calculateDiscount = (user, purchaseAmount) => {
  if (user.tier === "GOLD" && purchaseAmount > 200) {
    return 0.1;
  }
  // More conditions...
};
```

### Documentation Best Practices

1. **Keep It Updated**
   Documentation that's out of date is worse than no documentation. Update docs whenever you change code:

   ```javascript
   // Before change:
   /**
    * @param {string} name - User's name
    */
   const greetUser = (name) => {
     console.log(`Hello, ${name}!`);
   };

   // After adding feature:
   /**
    * @param {string} name - User's name
    * @param {string} [title] - Optional title (Mr., Mrs., etc.)
    */
   const greetUser = (name, title) => {
     const prefix = title ? `${title} ` : "";
     console.log(`Hello, ${prefix}${name}!`);
   };
   ```

2. **Use Consistent Formatting**
   Pick a documentation style and stick to it:

   ```javascript
   // Choose one style:

   // Style 1: JSDoc
   /**
    * @param {string} name
    * @returns {string}
    */

   // Style 2: Simple comments
   // Takes a name (string) and returns a greeting (string)

   // But don't mix them in the same project
   ```

3. **Include Examples**
   Real-world examples make documentation much more useful:

   ```javascript
   /**
    * Validates an email address.
    *
    * @param {string} email - The email to validate
    * @returns {boolean} Whether the email is valid
    *
    * @example
    * isValidEmail('user@example.com')  // Returns true
    * isValidEmail('invalid.email')     // Returns false
    * isValidEmail('')                  // Returns false
    */
   ```

4. **Document Errors and Edge Cases**
   Help others understand what could go wrong:

   ```javascript
   /**
    * Divides two numbers.
    *
    * @param {number} a - The dividend
    * @param {number} b - The divisor
    * @returns {number} The quotient
    *
    * @throws {Error} If divisor is zero
    * @throws {TypeError} If either parameter is not a number
    *
    * @example
    * try {
    *   divide(10, 2);  // Returns 5
    *   divide(10, 0);  // Throws Error: Cannot divide by zero
    * } catch (error) {
    *   console.error(error);
    * }
    */
   ```

Remember: Good documentation is like leaving a trail of breadcrumbs through your code. Future you (and others) will thank you for it!

Want to practice writing documentation for a specific piece of code? Or shall we explore more advanced documentation techniques?
