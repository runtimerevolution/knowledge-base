# Setting Up the Test Application

This guide walks you through setting up the Task Manager application that you'll use for testing exercises.

## Option A: React + Node.js Setup

### Backend Setup

```bash
# Create backend directory
mkdir task-manager && cd task-manager
mkdir backend && cd backend

# Initialize Node.js project
npm init -y

# Install dependencies
npm install express cors dotenv bcryptjs jsonwebtoken pg sequelize
npm install --save-dev nodemon jest supertest

# Create directory structure
mkdir -p src/{controllers,models,routes,middleware,config}
```

Create `backend/src/index.js`:

```javascript
const express = require('express');
const cors = require('cors');
require('dotenv').config();

const app = express();

app.use(cors());
app.use(express.json());

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/tasks', require('./routes/tasks'));
app.use('/api/projects', require('./routes/projects'));

const PORT = process.env.PORT || 3001;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

Create `backend/src/models/User.js`:

```javascript
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');
const bcrypt = require('bcryptjs');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4,
    primaryKey: true
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: { isEmail: true }
  },
  password: {
    type: DataTypes.STRING,
    allowNull: false
  },
  name: {
    type: DataTypes.STRING,
    allowNull: false
  }
}, {
  hooks: {
    beforeCreate: async (user) => {
      user.password = await bcrypt.hash(user.password, 10);
    }
  }
});

User.prototype.validatePassword = async function(password) {
  return bcrypt.compare(password, this.password);
};

module.exports = User;
```

Create `backend/src/models/Task.js`:

```javascript
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Task = sequelize.define('Task', {
  id: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4,
    primaryKey: true
  },
  title: {
    type: DataTypes.STRING,
    allowNull: false
  },
  description: {
    type: DataTypes.TEXT
  },
  status: {
    type: DataTypes.ENUM('todo', 'in_progress', 'done'),
    defaultValue: 'todo'
  },
  priority: {
    type: DataTypes.ENUM('low', 'medium', 'high'),
    defaultValue: 'medium'
  },
  dueDate: {
    type: DataTypes.DATE
  },
  userId: {
    type: DataTypes.UUID,
    allowNull: false
  },
  projectId: {
    type: DataTypes.UUID
  }
});

module.exports = Task;
```

### Frontend Setup

```bash
# From task-manager directory
npm create vite@latest frontend -- --template react-ts
cd frontend

# Install dependencies
npm install axios react-router-dom @tanstack/react-query
npm install --save-dev cypress @playwright/test
```

Create `frontend/src/pages/Login.tsx`:

```tsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { login } from '../services/auth';

export const Login: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    
    try {
      await login(email, password);
      navigate('/dashboard');
    } catch (err) {
      setError('Invalid email or password');
    }
  };

  return (
    <div className="login-container">
      <h1>Login</h1>
      <form onSubmit={handleSubmit} data-testid="login-form">
        {error && <div className="error" data-testid="error-message">{error}</div>}
        
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            type="email"
            id="email"
            data-testid="email-input"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
        </div>
        
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            type="password"
            id="password"
            data-testid="password-input"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
        </div>
        
        <button type="submit" data-testid="login-button">
          Login
        </button>
      </form>
    </div>
  );
};
```

Create `frontend/src/pages/Dashboard.tsx`:

```tsx
import React from 'react';
import { TaskList } from '../components/TaskList';
import { TaskForm } from '../components/TaskForm';

export const Dashboard: React.FC = () => {
  return (
    <div className="dashboard" data-testid="dashboard">
      <header>
        <h1>Task Manager</h1>
        <button data-testid="logout-button">Logout</button>
      </header>
      
      <main>
        <TaskForm />
        <TaskList />
      </main>
    </div>
  );
};
```

Create `frontend/src/components/TaskList.tsx`:

```tsx
import React from 'react';
import { useTasks } from '../hooks/useTasks';
import { TaskCard } from './TaskCard';

export const TaskList: React.FC = () => {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <div data-testid="loading">Loading...</div>;
  if (error) return <div data-testid="error">Error loading tasks</div>;

  return (
    <div className="task-list" data-testid="task-list">
      {tasks.length === 0 ? (
        <p data-testid="empty-message">No tasks yet. Create your first task!</p>
      ) : (
        tasks.map(task => (
          <TaskCard key={task.id} task={task} />
        ))
      )}
    </div>
  );
};
```

## Option B: Ruby on Rails Setup

```bash
# Create Rails application
rails new task-manager --database=postgresql -T
cd task-manager

# Add gems to Gemfile
cat >> Gemfile << 'EOF'
group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
end

group :test do
  gem 'capybara'
  gem 'selenium-webdriver'
  gem 'webdrivers'
  gem 'database_cleaner-active_record'
  gem 'cucumber-rails', require: false
end
EOF

bundle install

# Setup RSpec
rails generate rspec:install

# Setup Cucumber
rails generate cucumber:install

# Generate models
rails generate model User email:string:uniq password_digest:string name:string
rails generate model Project name:string description:text user:references
rails generate model Task title:string description:text status:integer priority:integer due_date:date user:references project:references

# Run migrations
rails db:create db:migrate
```

Create `app/models/user.rb`:

```ruby
class User < ApplicationRecord
  has_secure_password
  
  has_many :tasks, dependent: :destroy
  has_many :projects, dependent: :destroy
  
  validates :email, presence: true, uniqueness: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :name, presence: true
  validates :password, length: { minimum: 8 }, on: :create
end
```

Create `app/models/task.rb`:

```ruby
class Task < ApplicationRecord
  belongs_to :user
  belongs_to :project, optional: true
  
  enum status: { todo: 0, in_progress: 1, done: 2 }
  enum priority: { low: 0, medium: 1, high: 2 }
  
  validates :title, presence: true
  
  scope :by_status, ->(status) { where(status: status) }
  scope :by_priority, ->(priority) { where(priority: priority) }
  scope :due_soon, -> { where('due_date <= ?', 7.days.from_now) }
end
```

Create `app/controllers/tasks_controller.rb`:

```ruby
class TasksController < ApplicationController
  before_action :authenticate_user!
  before_action :set_task, only: [:show, :edit, :update, :destroy]

  def index
    @tasks = current_user.tasks
    @tasks = @tasks.by_status(params[:status]) if params[:status].present?
    @tasks = @tasks.by_priority(params[:priority]) if params[:priority].present?
  end

  def new
    @task = current_user.tasks.build
  end

  def create
    @task = current_user.tasks.build(task_params)
    
    if @task.save
      redirect_to tasks_path, notice: 'Task created successfully.'
    else
      render :new, status: :unprocessable_entity
    end
  end

  def update
    if @task.update(task_params)
      redirect_to tasks_path, notice: 'Task updated successfully.'
    else
      render :edit, status: :unprocessable_entity
    end
  end

  def destroy
    @task.destroy
    redirect_to tasks_path, notice: 'Task deleted successfully.'
  end

  private

  def set_task
    @task = current_user.tasks.find(params[:id])
  end

  def task_params
    params.require(:task).permit(:title, :description, :status, :priority, :due_date, :project_id)
  end
end
```

## Database Seeding

Create seed data for testing:

### Node.js (backend/src/seed.js)

```javascript
const User = require('./models/User');
const Task = require('./models/Task');
const Project = require('./models/Project');

async function seed() {
  // Create test user
  const user = await User.create({
    email: 'test@example.com',
    password: 'password123',
    name: 'Test User'
  });

  // Create projects
  const project = await Project.create({
    name: 'My First Project',
    description: 'A sample project',
    userId: user.id
  });

  // Create tasks
  const tasks = [
    { title: 'Complete onboarding', status: 'todo', priority: 'high' },
    { title: 'Review documentation', status: 'in_progress', priority: 'medium' },
    { title: 'Setup development environment', status: 'done', priority: 'high' },
  ];

  for (const task of tasks) {
    await Task.create({
      ...task,
      userId: user.id,
      projectId: project.id
    });
  }

  console.log('Database seeded successfully');
}

seed().catch(console.error);
```

### Rails (db/seeds.rb)

```ruby
# Clear existing data
Task.destroy_all
Project.destroy_all
User.destroy_all

# Create test user
user = User.create!(
  email: 'test@example.com',
  password: 'password123',
  name: 'Test User'
)

# Create admin user
admin = User.create!(
  email: 'admin@example.com',
  password: 'admin123',
  name: 'Admin User'
)

# Create projects
project = Project.create!(
  name: 'My First Project',
  description: 'A sample project',
  user: user
)

# Create tasks
[
  { title: 'Complete onboarding', status: :todo, priority: :high },
  { title: 'Review documentation', status: :in_progress, priority: :medium },
  { title: 'Setup development environment', status: :done, priority: :high },
  { title: 'Write first test', status: :todo, priority: :low },
  { title: 'Deploy to staging', status: :todo, priority: :medium, due_date: 3.days.from_now }
].each do |task_attrs|
  Task.create!(task_attrs.merge(user: user, project: project))
end

puts "Seeded #{User.count} users, #{Project.count} projects, #{Task.count} tasks"
```

## Running the Application

### React + Node.js

```bash
# Terminal 1: Start backend
cd backend
npm run dev

# Terminal 2: Start frontend
cd frontend
npm run dev
```

### Rails

```bash
# Start Rails server
rails server

# In another terminal, run tests
bundle exec rspec
bundle exec cucumber
```

## Verifying Setup

1. Open the application in your browser
2. Register a new user or login with test credentials
3. Create a task
4. Verify the task appears in the list

## Next Steps

Once your application is running, proceed to [Test Exercises](exercises.md) to start writing tests.
