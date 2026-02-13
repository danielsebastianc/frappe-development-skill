# Testing Patterns

Use when writing tests for Frappe applications with pytest and MagicMock.

## pytest Structure

### Basic Test Function

```python
import pytest
from unittest.mock import MagicMock, patch

def test_basic_insert():
    """Test document insertion"""
    doc = MagicMock()
    doc.description = "Test ToDo"
    doc.name = "TODO-001"
    
    assert doc.description == "Test ToDo"
    assert doc.name is not None
```

### pytest Fixtures

```python
import pytest
from unittest.mock import MagicMock

@pytest.fixture
def mock_doc():
    """Create a mock document"""
    doc = MagicMock()
    doc.name = "TEST-001"
    doc.description = "Test Description"
    return doc

def test_with_fixtures(mock_doc):
    """Test using fixtures"""
    assert mock_doc.name == "TEST-001"
```

## MagicMock Patterns

### Creating Mock Documents

```python
from unittest.mock import MagicMock

def test_with_mock_doc():
    """Create mock document with attributes"""
    doc = MagicMock()
    doc.name = "SI-001"
    doc.customer = "Customer A"
    
    # Mock child table
    item1 = MagicMock(item_code="ITEM-1", qty=5)
    doc.items = [item1]
    
    assert doc.customer == "Customer A"
    assert len(doc.items) == 1
```

### Mocking Methods

```python
def test_mock_method():
    """Mock document methods"""
    doc = MagicMock()
    doc.get_total.return_value = 500
    
    total = doc.get_total()
    assert total == 500
    doc.get_total.assert_called_once()
```

## Test Setup and Teardown

### Fixture-Based Setup

```python
import pytest
from unittest.mock import MagicMock, patch

@pytest.fixture
def setup_service():
    """Setup service with mocked dependencies"""
    with patch("my_app.repository.MyRepository") as mock_repo_class:
        mock_repo = mock_repo_class.return_value
        doc = MagicMock()
        service = MyService(doc)
        service.repo = mock_repo
        yield service, mock_repo

def test_service_operation(setup_service):
    """Test using setup fixture"""
    service, mock_repo = setup_service
    mock_repo.get_data.return_value = {"key": "value"}
    
    result = service.process()
    assert result == {"key": "value"}
```

## Patching Dependencies

### Module-Level Patching

```python
import pytest
from unittest.mock import patch, MagicMock

@pytest.fixture
def mock_repository():
    """Mock the repository dependency"""
    with patch("my_app.service.sales_service.SalesRepository") as MockRepo:
        repo = MockRepo.return_value
        yield repo

def test_with_patched_repo(mock_repository):
    """Test with mocked repository"""
    mock_repository.get_invoice.return_value = {"total": 1000}
    
    doc = MagicMock(name="SI-001")
    service = SalesService(doc)
    result = service.get_invoice_total()
    
    assert result == 1000
```

## Common Assertions

### Basic Assertions

```python
# Equality
assert doc.name == "TEST-001"
assert result == expected_value

# Collections
assert len(doc.items) == 3
assert "ITEM-1" in [item.item_code for item in doc.items]
```

### Exception Assertions

```python
import pytest
from unittest.mock import MagicMock

def test_validation_error():
    """Test that validation errors are raised"""
    service = MagicMock()
    service.validate.side_effect = ValueError("Invalid data")
    
    with pytest.raises(ValueError) as exc_info:
        service.validate()
    
    assert str(exc_info.value) == "Invalid data"
```

### Mock Call Assertions

```python
from unittest.mock import MagicMock, call

def test_mock_calls():
    """Verify mock interactions"""
    mock_db = MagicMock()
    
    mock_db.get_value("User", "test", "email")
    mock_db.set_value("User", "test", "status", "Active")
    
    # Assert specific call
    mock_db.get_value.assert_called_once_with("User", "test", "email")
```

## conftest.py Setup

Create `conftest.py` in your test directory to mock Frappe modules globally:

```python
# conftest.py
import sys
from unittest.mock import MagicMock

class MockDocument:
    def save(self):
        pass

mock_frappe_model_document = MagicMock()
mock_frappe_model_document.Document = MockDocument

def pytest_configure(config):
    """Setup mocking before test collection"""
    sys.modules["frappe"] = MagicMock()
    sys.modules["frappe.db"] = MagicMock()
    sys.modules["frappe.model"] = MagicMock()
    sys.modules["frappe.model.document"] = mock_frappe_model_document
    sys.modules["erpnext"] = MagicMock()

def pytest_unconfigure(config):
    """Cleanup after tests"""
    for module in ["frappe", "frappe.db", "erpnext"]:
        sys.modules.pop(module, None)
```

## Real-World Example: Service Test

```python
import pytest
from unittest.mock import MagicMock, patch
from my_app.service.sales_service import SalesService

@pytest.fixture
def mock_repository():
    with patch("my_app.service.sales_service.SalesRepository") as MockRepo:
        yield MockRepo.return_value

@pytest.fixture
def make_service():
    def _make(doc_data=None):
        doc = MagicMock()
        if doc_data:
            for key, value in doc_data.items():
                setattr(doc, key, value)
        return SalesService(doc)
    return _make

def test_process_sales_invoice(make_service, mock_repository):
    mock_repository.get_invoice.return_value = {"name": "SI-001", "total": 1000}
    
    doc = MagicMock()
    doc.name = "SI-001"
    service = SalesService(doc)
    service.repository = mock_repository
    
    result = service.process()
    assert result["total"] == 1000
```

## Running Tests

### pytest Commands

```bash
# Run all tests
python -m pytest

# Run specific test file
python -m pytest tests/test_sales_service.py

# Run with verbose output
python -m pytest -v

# Run with coverage
python -m pytest --cov=my_app --cov-report=html
```

### pytest Markers

```python
import pytest

@pytest.mark.unit
def test_small_unit():
    pass

@pytest.mark.slow
def test_slow_operation():
    pass
```

Configure in `pytest.ini`:
```ini
[pytest]
markers =
    unit: Unit tests (fast, isolated)
    slow: Tests that take > 1 second
```

## Quick Reference

| Pattern | Purpose | Example |
|---------|---------|---------|
| `@pytest.fixture` | Shared setup | `@pytest.fixture def mock_db():` |
| `MagicMock()` | Mock objects | `doc = MagicMock()` |
| `@patch()` | Mock dependencies | `@patch("module.function")` |
| `assert` | Validation | `assert result == expected` |
| `pytest.raises()` | Exception testing | `with pytest.raises(ValueError):` |
| `mock.assert_called()` | Verify calls | `mock_db.get.assert_called_once()` |

## Advanced Patterns

For complex scenarios see [testing-advanced.md](testing-advanced.md).
