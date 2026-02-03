# Advanced Testing Patterns

Use for complex testing scenarios, integration testing, and advanced pytest features.

## Mocking Complex Scenarios

### Mocking Multiple Dependencies

```python
import pytest
from unittest.mock import patch, MagicMock

@pytest.fixture
def mock_all_deps():
    """Mock multiple dependencies"""
    with patch("my_app.service.frappe.db.get_value") as mock_get_value, \
         patch("my_app.service.frappe.get_doc") as mock_get_doc:
        
        mock_get_value.return_value = "test_value"
        mock_get_doc.return_value = MagicMock()
        
        yield {
            "get_value": mock_get_value,
            "get_doc": mock_get_doc
        }

def test_multiple_mocks(mock_all_deps):
    """Test with multiple mocked dependencies"""
    service = MyService(MagicMock())
    service.process()
    mock_all_deps["get_value"].assert_called()
```

### Factory Fixtures

```python
@pytest.fixture
def make_service():
    """Factory for creating services with different docs"""
    def _make(doc_data=None):
        doc = MagicMock()
        if doc_data:
            for key, value in doc_data.items():
                setattr(doc, key, value)
        return SalesService(doc)
    return _make

def test_service_with_data(make_service):
    service = make_service({"name": "SI-001", "total": 1000})
    assert service.doc.name == "SI-001"
```

## Integration Testing

### Testing Service-Repository Integration

```python
def test_service_calls_repository():
    """Verify service properly delegates to repository"""
    with patch("my_app.service.OrderRepository") as mock_repo_class:
        mock_repo = mock_repo_class.return_value
        mock_repo.get_order.return_value = {"total": 100}
        
        service = OrderService(MagicMock())
        result = service.get_order_total("ORD-001")
        
        assert result == 100
        mock_repo.get_order.assert_called_once_with("ORD-001")
```

## Test Organization

### pytest.ini Configuration

```ini
[pytest]
markers =
    unit: Unit tests (fast, isolated)
    integration: Integration tests (may need DB)
    slow: Tests that take > 1 second
```

### Running with Markers

```bash
# Run only unit tests
python -m pytest -m "unit"

# Skip slow tests
python -m pytest -m "not slow"
```

## Advanced Assertions

### Asserting Not Called

```python
from unittest.mock import MagicMock

def test_no_unexpected_calls():
    mock_db = MagicMock()
    process_read_only()
    mock_db.delete.assert_not_called()
```

### Verifying Call Sequences

```python
from unittest.mock import call

def test_call_sequence():
    mock_db = MagicMock()
    
    mock_db.get_value("User", "test", "email")
    mock_db.get_value("Role", "test", "name")
    
    expected_calls = [
        call("User", "test", "email"),
        call("Role", "test", "name")
    ]
    mock_db.get_value.assert_has_calls(expected_calls)
```

## Testing DocType Controllers

### Testing Document Lifecycle

```python
def test_before_submit_validation():
    """Test validation in before_submit hook"""
    doc = MagicMock()
    doc.total = -100  # Invalid negative total
    
    with pytest.raises(Exception) as exc_info:
        service = InvoiceService(doc)
        service.validate_before_submit()
    
    assert "negative" in str(exc_info.value).lower()
```

## Performance Testing

### Timing Tests

```python
import time

def test_performance():
    """Test that operation completes within time limit"""
    start = time.time()
    
    service = MyService(MagicMock())
    service.process_large_dataset()
    
    elapsed = time.time() - start
    assert elapsed < 5.0  # Must complete in 5 seconds
```

## Test Data Management

### Fixture Scopes

```python
@pytest.fixture(scope="session")
def db_connection():
    """Create once per test session"""
    conn = create_connection()
    yield conn
    conn.close()

@pytest.fixture(scope="function")
def clean_db(db_connection):
    """Clean before each test function"""
    db_connection.truncate()
    yield db_connection
```

## Debugging Tests

### pytest Debugging

```bash
# Stop on first failure
python -m pytest -x

# Show local variables on failure
python -m pytest -l

# Enter debugger on failure
python -m pytest --pdb

# Show full traceback
python -m pytest --tb=long
```
