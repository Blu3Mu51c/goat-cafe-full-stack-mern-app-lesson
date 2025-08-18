# Goat Cafe Vite - Full-Stack E-commerce Application

<div style="display: flex; justify-content: space-between;">

<p><img width="862" height="342" alt="Untitled Diagram drawio" src="https://github.com/user-attachments/assets/a3d67539-f43f-4923-a6f5-7dfce418b427" /></p>
<p><img src="https://i.imgur.com/TGZKfoI.png" alt="components"></p>

</div>

## 🎯 Project Overview
A complete full-stack e-commerce application built with React (Vite), Express.js, and MongoDB. This project demonstrates modern web development practices including JWT authentication, RESTful APIs, and responsive design.

## 🚀 Quick Start

### Prerequisites
- Node.js (v16 or higher)
- npm or yarn
- MongoDB Atlas account (or local MongoDB)

### Installation
```bash
# Clone the repository
git clone <your-repo-url>
cd goat-cafe-vite

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your MongoDB URI and JWT secret

# Start development servers
npm run dev        # Frontend (Vite)
npm run server     # Backend (Express)
```

## 📚 **Complete Course Available!**

This project includes a comprehensive, step-by-step course that teaches you how to build this full-stack application from scratch. 

### 🎓 **Course Location: `FULL_STACK_COURSE/` folder**

**Start here:** [FULL_STACK_COURSE/README.md](./FULL_STACK_COURSE/README.md)

### 📖 **Course Structure**
The course is organized into 9 numbered lessons plus additional explanatory files:

#### **Core Lessons (Follow in Order)**
1. [Project Setup and Configuration](./FULL_STACK_COURSE/01_PROJECT_SETUP_AND_CONFIGURATION.md)
2. [Backend Foundation and Server Setup](./FULL_STACK_COURSE/02_BACKEND_FOUNDATION_AND_SERVER_SETUP.md)
3. [JWT Authentication Implementation](./FULL_STACK_COURSE/03_JWT_AUTHENTICATION_IMPLEMENTATION.md)
4. [Database Models and Mongoose Schemas](./FULL_STACK_COURSE/04_DATABASE_MODELS_AND_MONGOOSE_SCHEMAS.md)
5. [Controllers and API Routes](./FULL_STACK_COURSE/05_CONTROLLERS_AND_API_ROUTES.md)
6. [Frontend Setup with Vite and React](./FULL_STACK_COURSE/06_FRONTEND_SETUP_WITH_VITE_AND_REACT.md)
7. [Utilities and Services Layer](./FULL_STACK_COURSE/07_UTILITIES_AND_SERVICES_LAYER.md)
8. [Component State Logic and React Hooks](./FULL_STACK_COURSE/08_COMPONENT_STATE_LOGIC.md)
9. [End-to-End Flow: Complete Application Walkthrough](./FULL_STACK_COURSE/09_END_TO_END_FLOW.md)

#### **Additional Resources**
- [Code Explanations](./FULL_STACK_COURSE/CODE_EXPLANATIONS.md) - Line-by-line backend code explanations
- [Utility Logic Explanation](./FULL_STACK_COURSE/UTILITY_LOGIC_EXPLANATION.md) - Frontend service layer details
- [Component State Logic](./FULL_STACK_COURSE/COMPONENT_STATE_LOGIC.md) - React state management patterns
- [Debug Guide](./FULL_STACK_COURSE/DEBUG_GUIDE.md) - Comprehensive troubleshooting

## 🏗️ Project Architecture

### Backend
- **Express.js** - Web framework
- **MongoDB + Mongoose** - Database and ODM
- **JWT** - Authentication
- **bcrypt** - Password hashing
- **CORS** - Cross-origin resource sharing

### Frontend
- **React 18** - UI library
- **Vite** - Build tool and dev server
- **SCSS** - CSS preprocessor
- **CSS Modules** - Scoped styling
- **React Router** - Client-side routing

### Key Features
- ✅ User authentication (signup/login)
- ✅ JWT token management
- ✅ Product catalog with filtering
- ✅ Shopping cart functionality
- ✅ Order management system
- ✅ Responsive design
- ✅ Protected API routes

## 🧪 Testing

### Backend Testing
```bash
# Test API endpoints
curl -X POST http://localhost:3000/api/users/signup \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"password123"}'
```

### Frontend Testing
- Open browser developer tools
- Check network requests
- Verify authentication flow
- Test responsive design

## 🐛 Troubleshooting

If you encounter issues:

1. **Check the Debug Guide**: [FULL_STACK_COURSE/DEBUG_GUIDE.md](./FULL_STACK_COURSE/DEBUG_GUIDE.md)
2. **Review Code Explanations**: [FULL_STACK_COURSE/CODE_EXPLANATIONS.md](./FULL_STACK_COURSE/CODE_EXPLANATIONS.md)
3. **Verify your setup** against the course lessons
4. **Check console errors** for specific messages

## 📁 Project Structure

```
goat-cafe-vite/
├── FULL_STACK_COURSE/          # 📚 Complete course materials
├── src/                        # Frontend React components
├── routes/                     # Backend API routes
├── controllers/                # Backend business logic
├── models/                     # MongoDB schemas
├── config/                     # Configuration files
├── public/                     # Static assets
├── server.js                   # Backend entry point
├── app-server.js              # Express app configuration
├── vite.config.js             # Vite configuration
└── package.json               # Dependencies and scripts
```

## 🚀 Production Deployment

For production deployment information, see: [PRODUCTION_DEPLOYMENT.md](./PRODUCTION_DEPLOYMENT.md)

## 🤝 Contributing

This project serves as both a working application and a comprehensive learning resource. Feel free to:

- Study the code and course materials
- Build upon the foundation
- Suggest improvements
- Share your learning experience

## 📄 License

This project is provided for educational purposes. Use the code examples to learn and build your own applications.

## 🎉 Get Started!

**Ready to learn full-stack development?** 

1. **Start with the course**: [FULL_STACK_COURSE/README.md](./FULL_STACK_COURSE/README.md)
2. **Follow the lessons in order** (01-09)
3. **Build along with the course**
4. **Reference the additional resources** when you need deeper understanding

**Happy coding! 🚀**
