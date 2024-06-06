import React, { useState } from 'react';
import { Dropdown, DropdownButton,Modal,Button } from 'react-bootstrap';
import 'bootstrap/dist/css/bootstrap.min.css';
import './App.css';
import { XCircle } from 'lucide-react';

function App() {
  const [data, setData] = useState([]);
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [age, setAge] = useState(0);
  const [depName, setDepName] = useState('');
  const [group, setGroup] = useState('');
  const [id, setId] = useState(0);
  const [isUpdate, setIsUpdate] = useState(false);
  const [IDCounter, setIDCounter] = useState(1);
  const [depCodeCounter, setDepCodeCounter] = useState(1);
  const [detailsId, setDetailsId] = useState(null);
  const[showModal,setShowModal] = useState(false);
  const[selectedItem,setSelectedItem] = useState(null);

  const handleEdit = (id) => {
    const item = data.find(item => item.id === id);
    if (item) {
      setIsUpdate(true);
      setId(id);
      setFirstName(item.firstName || '');
      setLastName(item.lastName || '');
      setAge(item.Age || 0);
      setDepName(item.depName || '');
      setGroup(item.group || '');
    }
  };

  const handleDelete = (id) => {
    if (window.confirm("Are you sure you want to delete this item?")) {
      const updatedData = data.filter(item => item.id !== id);
      setData(updatedData);
    }
  };

  const handleDetails = (id) => {
   const item = data.find(item => item.id === id);
   if(item){
    setShowModal(true);
    setSelectedItem(item);
   }
  };

  const handleSave = (e) => {
    e.preventDefault();
    let error = '';

    const nameRegex = /^[a-zA-Z ]+$/;

    if (!firstName.match(nameRegex) && !depName.match(nameRegex))
      error += 'Either First name or Department name is required and should contain only alphabets, ';

    if (!lastName.match(nameRegex) && !group)
      error += 'Either Last name or Group is required and should contain only alphabets, ';

    if (age <= 0 && !depName)
      error += 'Age is required if entering employee data.';

    if (data.some(item => item.depName === depName && item.group === group)) {
      error += 'The same group name cannot be used again in the same department.';
    }

    if (error === '') {
      const newObject = {
        id: IDCounter,
        firstName,
        lastName,
        Age: age,
        depName,
        group,
        depcode: `Dep@${depCodeCounter}`
      };
      setData([...data, newObject]);
      setIDCounter(IDCounter + 1);
      setDepCodeCounter(depCodeCounter + 1);
      handleClear();
    } else {
      alert(error);
    }
  };

  const handleUpdate = () => {
    let error = '';

    if (data.some(item => item.id !== id && item.depName === depName && item.group === group)) {
      error += 'The same group name cannot be used again in the same department.';
    }

    if (error === '') {
      const index = data.findIndex(item => item.id === id);
      const updatedData = [...data];
      updatedData[index] = { ...updatedData[index], firstName, lastName, Age: age, depName, group };
      setData(updatedData);
      handleClear();
    } else {
      alert(error);
    }
  };

  const handleClear = () => {
    setId(0);
    setFirstName('');
    setLastName('');
    setAge(0);
    setDepName('');
    setGroup('');
    setIsUpdate(false);
  };

  return (
    <div className="App">
      <h1>Custom Fields</h1>
      <div className="form-container">
        <label>
          Name:
          <input type='text' placeholder='Enter Label' onChange={(e) => setFirstName(e.target.value)} value={firstName} />
        </label>
        <label>
          Age:
          <input type='number' placeholder='Enter Age' onChange={(e) => setAge(Number(e.target.value))} value={age} />
        </label>
        <label>
          Department Name:
          <input type='text' placeholder='Enter Department Name' onChange={(e) => setDepName(e.target.value)} value={depName} />
        </label>
        <label>
          Created By:
          <input type='text' placeholder='Enter Name' onChange={(e) => setGroup(e.target.value)} value={group} />
        </label>
        <div className="form-buttons">
          {
            !isUpdate ?
              <button className='btn btn-primary' onClick={(e) => handleSave(e)}>Save</button>
              :
              <button className='btn btn-primary' onClick={handleUpdate}>Update</button>
          }
          <button className='btn btn-danger' onClick={handleClear}>Clear</button>
        </div>
      </div>

      <table className='table table-hover'>
        <thead>
          <tr>
            <td>Field ID</td>
            <td>ID</td>
            <td>First Name</td>
            <td></td>
            <td>Age</td>
            <td>Department Code</td>
            <td>Department Name</td>
            <td>Group</td>
            <td>Actions</td>
          </tr>
        </thead>
        <tbody>
          {
            data.map((item, index) => (
              <React.Fragment key={index}>
                <tr>
                  <td>{index + 1}</td>
                  <td>{item.id}</td>
                  <td>{item.firstName}</td>
                  <td>{item.lastName}</td>
                  <td>{item.Age}</td>
                  <td>{item.depcode}</td>
                  <td>{item.depName}</td>
                  <td>{item.group}</td>
                  <td>
                    <DropdownButton id="dropdown-basic-button" title="⋮">
                      <Dropdown.Item onClick={() => handleEdit(item.id)}>Edit</Dropdown.Item>
                      <Dropdown.Item onClick={() => handleDelete(item.id)}>Delete</Dropdown.Item>
                      <Dropdown.Item onClick={() => handleDetails(item.id)}>Details</Dropdown.Item>
                    </DropdownButton>
                  </td>
                </tr>
                {detailsId === item.id && (
                  <tr>
                    <td colSpan="9">
                      <div className="details-container">
                        <button onClick={() => setDetailsId(null)}><XCircle /></button>
                        <div>
                          <p>First Name: {item.firstName}</p>
                          <p>Last Name: {item.lastName}</p>
                          <p>Age: {item.Age}</p>
                          <p>Department Code: {item.depcode}</p>
                          <p>Department Name: {item.depName}</p>
                          <p>Created By: {item.group}</p>
                        </div>
                      </div>
                    </td>
                  </tr>
                )}
              </React.Fragment>
            ))
          }
        </tbody>
      </table>
      <Modal show={showModal} onHide={() => setShowModal(false)}>
        <Modal.Header closeButton>
          <Modal.Title>Employee Details</Modal.Title>
        </Modal.Header>
        <Modal.Body>
          {selectedItem && (
            <div>
                <p><strong>First Name : </strong>{selectedItem.firstName}</p>
                <p><strong>Last Name : </strong>{selectedItem.lastName}</p>
                <p><strong>Age : </strong>{selectedItem.Age}</p>
                <p><strong>Department Code : </strong>{selectedItem.depcode}</p>
                <p><strong>Department Name : </strong>{selectedItem.depName}</p>
                <p><strong>Group : </strong>{selectedItem.group}</p>
            </div>
          )}
        </Modal.Body>
      </Modal>
     
    </div>
  );
}

export default App;
