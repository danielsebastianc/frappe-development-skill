# Frappe Development Agent Skill

Personal comprehensive skill for developing custom Frappe/ERPNext applications with Controller-Service-Repository (CSR) architecture, best practices, and testing patterns.

## Description

This skill is designed for **AI agents and coding assistants** (like Opencode) to provide expert guidance when developing Frappe/ERPNext applications. It offers comprehensive knowledge for building robust Frappe applications using clean architecture principles, covering DocType controllers, database queries, error handling, and testing strategies.

## Topics Covered

- **Architecture Patterns** - CSR architecture implementation, API controllers, services, and repositories
- **DocType Controllers** - Lifecycle management, validation methods, document flags, and child tables
- **Database Queries** - Query patterns including `get_doc`, `get_list`, `frappe.qb`, and filters
- **Error Handling** - Proper error handling with `frappe.throw()`, exceptions, and logging strategies
- **Testing** - Unit and integration testing with pytest and MagicMock
- **Advanced Testing** - Complex testing scenarios and mocking strategies

## When to Use

AI agents should load this skill when working on:
- Building custom DocType controllers
- Implementing business logic with CSR architecture
- Creating API endpoints with `@frappe.whitelist()`
- Querying the Frappe database
- Implementing validations and error handling
- Writing tests for Frappe applications
- Extending standard ERPNext DocTypes

## File Structure

```
frappe-development-skill/
├── SKILL.md                    # Main skill metadata and workflows
├── README.md                   # This documentation file
└── patterns/                   # Skill pattern guides
    ├── architecture-patterns.md      # CSR architecture guide
    ├── doctype-controllers.md      # DocType controller patterns
    ├── database-queries.md         # Database query patterns
    ├── error-handling.md           # Error handling strategies
    ├── testing-patterns.md         # Basic testing guide
    └── testing-advanced.md         # Advanced testing scenarios
```

## Compatibility

This skill is designed for **Frappe Framework v15 and v16** and **ERPNext v15 and v16**.

## Contributing

Contributions are welcome! Here's how you can help improve this skill:

### Submitting Changes

1. **Fork** the repository
2. **Create a branch** for your changes: `git checkout -b feature/your-feature-name`
3. **Make your changes** following the existing style and structure
4. **Test** your changes with opencode to ensure they work correctly
5. **Commit** with a clear message: `git commit -m "Add: description of change"`
6. **Push** to your fork: `git push origin feature/your-feature-name`
7. **Open a Pull Request** with a detailed description

### Contribution Guidelines

- Keep content focused on Frappe/ERPNext development best practices
- Follow the existing markdown structure and formatting
- Ensure code examples (if any) are tested and working
- Update relevant sections if adding new topics
- Maintain compatibility with Frappe v15 and v16

### Areas for Contribution

- Additional patterns and workflows
- More testing scenarios
- Performance optimization tips
- Integration guides with other Frappe apps
- Common pitfalls and solutions

## License

MIT License - Feel free to use, modify, and distribute this skill.

## Author

Created and maintained by [danielsebastianc](https://github.com/danielsebastianc)

## Acknowledgments

This skill is built based on Frappe Framework best practices and community knowledge. Special thanks to the Frappe and ERPNext community for their continuous contributions to the ecosystem.
