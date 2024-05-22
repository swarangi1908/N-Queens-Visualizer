# N-Queen Visualiser

- The N-Queens puzzle is the problem of placing N chess queens on an N×N chessboard so that no two queens threaten each other. Thus, a solution requires that no two queens share the same row, column, or diagonal.

- This algorithm is designed using recursion.

![N-Queen-visualisation](visualisation.gif)

**<p align='center'>You can find the website live <a href="https://nqueen.netlify.app/">here</a></p>**
// Import necessary modules
import { render, screen, fireEvent } from '@testing-library/react';
import { Provider } from 'react-redux';
import configureStore from 'redux-mock-store';
import { vi } from 'vitest';
import SearchComponent from './SearchComponent'; // Adjust the import to match your file structure

// Create a mock store
const mockStore = configureStore([]);
const initialState = {
  // Define your initial state here
  searchRes: [],
  searchObject: {
    instrumentSearch: {
      instrumentIdentifierType: '',
      identifierValue: ''
    }
  }
};

describe('SearchComponent', () => {
  it('should update searchObject state correctly when handleChange is called', () => {
    // Initialize mock store with initial state
    const store = mockStore(initialState);

    // Render the component with Provider
    const { getByLabelText } = render(
      <Provider store={store}>
        <SearchComponent />
      </Provider>
    );

    // Get the select element and text field
    const selectElement = getByLabelText('Identifier');
    const textField = screen.queryByPlaceholderText('Security Search');

    // Simulate changing the select element
    fireEvent.change(selectElement, { target: { name: 'instrumentIdentifierType', value: 'ACCOUNTS' } });

    // Verify the select value is updated
    expect(selectElement.value).toBe('ACCOUNTS');

    // Verify the text field appears when instrumentIdentifierType is "ACCOUNTS"
    expect(screen.queryByPlaceholderText('Security Search')).toBeInTheDocument();

    // Simulate changing the text field value
    if (textField) {
      fireEvent.change(textField, { target: { name: 'identifierValue', value: '12345' } });
    }

    // Verify the text field value is updated
    expect(textField.value).toBe('12345');
  });
});