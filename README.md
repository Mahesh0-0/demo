import React, { useState, useEffect } from "react";

interface Product {
  title: string;
}

interface CartItem {
  product: Product;
  price: number;
}

const ProductForm: React.FC = () => {
  const [products, setProducts] = useState<Product[]>([]);
  const [selectedProduct, setSelectedProduct] = useState<string>("");
  const [price, setPrice] = useState<number>(0);
  const [cartItems, setCartItems] = useState<CartItem[]>([]);

  useEffect(() => {
    // Simulating reading the JSON and setting the products
    const productsJson = '[{"title":"Product1"},{"title":"Product2"},{"title":"Product3"}]';
    setProducts(JSON.parse(productsJson));
  }, []);

  const handleProductChange = (event: React.ChangeEvent<HTMLSelectElement>) => {
    setSelectedProduct(event.target.value);
  };

  const handlePriceChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setPrice(Number(event.target.value));
  };

  const handleAddToCart = () => {
    if (selectedProduct && price > 0) {
      const newCartItem: CartItem = {
        product: { title: selectedProduct },
        price: price
      };
      setCartItems([...cartItems, newCartItem]);
      setSelectedProduct("");
      setPrice(0);
    }
  };

  return (
    <div>
      <div>
        <select value={selectedProduct} onChange={handleProductChange}>
          <option value="">Select a product</option>
          {products.map((product, index) => (
            <option key={index} value={product.title}>
              {product.title}
            </option>
          ))}
        </select>
        <input type="number" value={price} onChange={handlePriceChange} placeholder="Price" />
        <button onClick={handleAddToCart}>Add to Cart</button>
      </div>
      <div>
        <h3>Cart</h3>
        <ul>
          {cartItems.map((item, index) => (
            <li key={index}>
              {item.product.title} - ${item.price}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
};

export default ProductForm;





import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const rootElement = document.getElementById("root")!;
const root = ReactDOM.createRoot(rootElement);

root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);





import React from "react";
import ProductForm from "./productForm";

function App() {
  return (
    <div>
      <h1>Product Cart</h1>
      <ProductForm />
    </div>
  );
}

export default App;









import React, { useState, useRef } from "react";

const Stopwatch: React.FC = () => {
  const [isRunning, setIsRunning] = useState(false);
  const [time, setTime] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  const startStopwatch = () => {
    if (!isRunning) {
      intervalRef.current = setInterval(() => {
        setTime((prevTime) => prevTime + 1);
      }, 1000);
    } else {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    }
    setIsRunning(!isRunning);
  };

  const resetStopwatch = () => {
    setTime(0);
    if (isRunning && intervalRef.current) {
      clearInterval(intervalRef.current);
      setIsRunning(false);
    }
  };

  return (
    <div>
      <h1>Stopwatch</h1>
      <div>Time: {time}</div>
      <button onClick={startStopwatch}>{isRunning ? "Stop" : "Start"}</button>
      <button onClick={resetStopwatch}>Reset</button>
    </div>
  );
};

export default Stopwatch;





import React from "react";

interface Product {
  title: string;
  price: number;
}

interface CartItem {
  product: Product;
  id: number;
}

interface CartState {
  cartItems: CartItem[];
  isLoading: boolean;
}

class Cart extends React.Component<{}, CartState> {
  constructor(props: {}) {
    super(props);
    this.state = {
      cartItems: [],
      isLoading: true,
    };
  }

  componentDidMount() {
    this.fetchCartItems();
  }

  fetchCartItems = () => {
    fetch("https://www.baljeetmatta.com/codequotient/Questions/getJSON")
      .then((response) => response.json())
      .then((data) => {
        const cartItems: CartItem[] = data.map((item: any, index: number) => ({
          product: item,
          id: index,
        }));
        this.setState({ cartItems, isLoading: false });
      })
      .catch((error) => {
        console.error("Error fetching cart items:", error);
      });
  };

  removeItem = (id: number) => {
    const updatedCartItems = this.state.cartItems.filter(
      (item) => item.id !== id
    );
    this.setState({ cartItems: updatedCartItems });
  };

  render() {
    const { cartItems, isLoading } = this.state;

    if (isLoading) {
      return <div>Loading...</div>;
    }

    return (
      <div>
        <h1>Cart Items</h1>
        {cartItems.map((cartItem) => (
          <div key={cartItem.id}>
            <p>{cartItem.product.title} - ${cartItem.product.price}</p>
            <button onClick={() => this.removeItem(cartItem.id)}>
              Remove
            </button>
          </div>
        ))}
      </div>
    );
  }
}

export default Cart;




class Transaction {
  private _value: number;

  constructor(value: number) {
    this._value = value;
  }

  value(): number {
    return this._value;
  }
}

class Account {
  private _accno: number;

  constructor(accno: number) {
    this._accno = accno;
  }

  process(trans: Transaction): boolean {
    console.log(`Processing transaction with value: ${trans.value()} for account: ${this._accno}`);
    // Perform actual processing logic here
    return true;
  }
}

class FilteredAccount extends Account {
  private _filteredCount: number;

  constructor(accno: number) {
    super(accno);
    this._filteredCount = 0;
  }

  process(trans: Transaction): boolean {
    if (trans.value() !== 0) {
      return super.process(trans);
    } else {
      this._filteredCount++;
      console.log(`Filtered out transaction with zero value for account: ${this._accno}`);
      return true;
    }
  }

  filtered(): number {
    return this._filteredCount;
  }
}

// Example usage
const acc = new FilteredAccount(123);
const trans1 = new Transaction(100);
const trans2 = new Transaction(0);
const trans3 = new Transaction(-50);

acc.process(trans1); // Process non-zero transaction
acc.process(trans2); // Filter out zero-valued transaction
acc.process(trans3); // Process non-zero transaction

console.log(`Number of filtered transactions: ${acc.filtered()}`);
  
  
  
  
  
  interface Todo {
    id: number;
    text: string;
    completed: boolean;
}

const todos: Todo[] = [];

function addTodo() {
    const input = document.getElementById('todo-input') as HTMLInputElement;
    const text = input.value.trim();

    if (text !== '') {
        const newTodo: Todo = {
            id: todos.length + 1,
            text: text,
            completed: false,
        };

        todos.push(newTodo);
        input.value = '';
        renderTodos();
    }
}

function toggleTodoStatus(id: number) {
    const todo = todos.find(todo => todo.id === id);

    if (todo) {
        todo.completed = !todo.completed;
        renderTodos();
    }
}

function removeTodo(id: number) {
    const index = todos.findIndex(todo => todo.id === id);

    if (index !== -1) {
        todos.splice(index, 1);
        renderTodos();
    }
}

function renderTodos() {
    const todoList = document.getElementById('todo-list');
    todoList.innerHTML = '';

    todos.forEach(todo => {
        const li = document.createElement('li');
        const span = document.createElement('span');
        const removeBtn = document.createElement('button');

        span.textContent = todo.text;
        span.addEventListener('click', () => toggleTodoStatus(todo.id));

        removeBtn.textContent = 'Remove';
        removeBtn.addEventListener('click', () => removeTodo(todo.id));

        li.appendChild(span);
        li.appendChild(removeBtn);
        todoList.appendChild(li);

        if (todo.completed) {
            span.classList.add('completed');
        }
    });
}

document.getElementById('add-button').addEventListener('click', addTodo);

  
 class TollBooth {
    private car: number;
    private cash: number;
    constructor() {
        this.car = 0;
        this.cash = 0;
    }

    public pay(): void {
        this.car++;
        this.cash += 50;
    }

    public notpay(): void {
        this.car++;
    }

    public display(): void {
        console.log(`Total cars: ${this.car}`);
        console.log(`Total cash collected: ${this.cash} Rupees`);
    }
}
function sample(inputs: string[]): void {
    const tollBooth = new TollBooth();

    inputs.forEach((input) => {
        if (input === 'p') {
            tollBooth.pay();
        } else if (input === 'n') {
            tollBooth.notpay();
        }
    });

    tollBooth.display();
}


sample(['p', 'p', 'n', 'p', 'n']);
sample(['p', 'p', 'n', 'p', 'p', 'n', 'p']);
sample(['p', 'p', 'n', 'p', 'n', 'p', 'p', 'p', 'n']);
