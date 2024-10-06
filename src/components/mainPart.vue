<template>
    <div class="sidebar">
        <div v-for="eachCabinet in cabinetNamesList" :key="eachCabinet.id" class="eachCabinetButton">
            <button class="cabinetButton" @click="toggleComputerss(eachCabinet.name)">{{ eachCabinet.name }}</button>
            <button class="addItemInCabinetButton" @click="showComputerForm(eachCabinet.name)">Добавить оборудование</button>
            <button class="deleteCabinetButton" @click="deleteCabinet(eachCabinet.name)">x</button>
            <div class="computer-list" :id="`computers-` + eachCabinet.name" style="display: none;"></div>
        </div>
        <div>
            <button id="addCabinetButton" @click="addCabinet">Добавить кабинет</button>
            <button id="toMainPage" @click="showMainPage()">На главную</button>
        </div>
    </div>
    <div class="mainGuidePage">
        <div class="top_main_info">
                <h1 id="main_header">Управление оборудованием.</h1>
            </div>
                        <div class="main_info">
                            <div class="instructions">
                                <h2>Как добавить кабинет через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>На боковой панели нажать на кнопку "Добавить кабинет".</li>
                                        <li>Выбрать название кабинета.</li>
                                        <li>Нажать "Ок".</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как добавить оборудование через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>На боковой панели выбрать кабинет, в который нужно добавить обоурдование.</li>
                                        <li>В открывшейся форме выбрать название оборудования и его тип.</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как редактировать характеристики оборудования через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>В боковой панели выбрать нужный кабинет.</li>
                                        <li>Там же выбрать нужное оборудование. (После изменения нужно нажать на кнопку "Сохранить характеристики", иначе изменения не сохранятся.)</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как добавить оборудование через add_item.py (Нужен установленный питон.): </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>Открыть файл add_item.py.</li>
                                        <li>Ввести название кабинета, в который нужно добавить оборудование и нажать "Enter".</li>
                                        <li>Ввести инвентарный номер и нажать "Enter".</li>
                                    </ol>
                                </div>
                            </div>
                        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            cabinetNamesList: []
        };
    },
    methods: {
        async getCabinets() {
            this.cabinetNamesList = await fetchCabinets();
        },
        async addCabinet() {
            const cabinetName = prompt("Введите название кабинета:");
            if (!cabinetName) return;

            const response = await fetch('http://127.0.0.1:5001/add_cabinet', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ name: cabinetName })
            });

            if (response.status === 204) {
                await this.getCabinets(); 
            } else {
                alert("Кабинет с таким названием уже существует.");
            }
        },
        toggleComputerss(cabinetName) {
            return toggleComputers(cabinetName)
        },
        async deleteCabinet(cabinetName) {
            if (!confirm(`Вы уверены, что хотите удалить кабинет "${cabinetName}"? Это действие удалит все оборудование и характеристики.`)) {
        return;}
        const response = await fetch(`http://127.0.0.1:5001/delete_cabinet/${cabinetName}`, {
            method: 'DELETE'});
        if (response.ok) {
            alert(`Кабинет "${cabinetName}" был успешно удален.`);
            await this.getCabinets();
        } else {
            alert('Ошибка при удалении кабинета.');
        }
        },
        showComputerForm(cabinetName) {
            const contentDiv = document.querySelector('.mainGuidePage');
            contentDiv.innerHTML = `
                <h2>Добавить оборудование в кабинет ${cabinetName}.</h2>
                <form id="add-computer-form">
                    <div class='oborydovanie-form'>
                        <label for="name" id='obor-name'>Название оборудования:</label><br>
                        <input type="text" id="name" name="name" required><br>
                    </div>
                    <div class='oborydovanie-form'>
                        <label for="type" id='obor-type'>Тип оборудования:</label><br>
                        <select id="type" name="type">
                            <option value="comp">Компьютер</option>
                            <option value="laptop">Ноутбук</option>
                            <option value="monitor">Монитор</option>
                            <option value="projector">Проектор</option>
                            <option value="printer">Принтер</option>
                            <option value="monoblock">Моноблок</option>
                            <option value="interactive_board">Интерактивная доска</option>
                        </select><br>
                    </div>

                    <button type="submit" class="add-computer-btn">Добавить оборудование</button>
                </form>
            `;

            document.getElementById('add-computer-form').addEventListener('submit', async function (event) {
                event.preventDefault();
                const name = document.getElementById('name').value;
                const type = document.getElementById('type').value;

                await fetch('http://127.0.0.1:5001/add_computer', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ cabinet_name: cabinetName, name, type })
                }).then(response => {
                    if (response.status === 204) {
                        fetchCabinets();
                    } else {
                        alert("Оборудование с таким названием уже существует.");
                    }
                });
                loadComputers(cabinetName);
            });
            },
        showMainPage() {
            const contendDiv = document.querySelector('.mainGuidePage');
            contendDiv.innerHTML = `
            <div class="top_main_info">
                <h1 id="main_header">Управление оборудованием.</h1>
            </div>
                        <div class="main_info">
                            <div class="instructions">
                                <h2>Как добавить кабинет через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>На боковой панели нажать на кнопку "Добавить кабинет".</li>
                                        <li>Выбрать название кабинета.</li>
                                        <li>Нажать "Ок".</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как добавить оборудование через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>На боковой панели выбрать кабинет, в который нужно добавить обоурдование.</li>
                                        <li>В открывшейся форме выбрать название оборудования и его тип.</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как редактировать характеристики оборудования через сайт: </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>В боковой панели выбрать нужный кабинет.</li>
                                        <li>Там же выбрать нужное оборудование. (После изменения нужно нажать на кнопку "Сохранить характеристики", иначе изменения не сохранятся.)</li>
                                    </ol>
                                </div>
                            </div>
                            <div class="instructions">
                                <h2>Как добавить оборудование через add_item.py (Нужен установленный питон.): </h2>
                                <div class="instruction_items">
                                    <ol>
                                        <li>Открыть файл add_item.py.</li>
                                        <li>Ввести название кабинета, в который нужно добавить оборудование и нажать "Enter".</li>
                                        <li>Ввести инвентарный номер и нажать "Enter".</li>
                                    </ol>
                                </div>
                            </div>
                        </div>
                    `;
        },


    },
    mounted() {
        this.getCabinets();
    }
};

function toggleComputers(cabinetName) {
            const computerList = document.getElementById(`computers-${cabinetName}`);
            if (computerList.style.display === 'none') {
                computerList.style.display = 'block';
                // console.log('Вызвана функция toggleComputers с кабинетом: ' + cabinetName);
                loadComputers(cabinetName); // Загружаем список оборудования при первом открытии
            } else {
                computerList.style.display = 'none';
            }
        }


async function loadComputers(cabinetName) {
    const response = await fetch(`http://127.0.0.1:5001/get_computers/${cabinetName}`);
    const computers = await response.json();
    const computerList = document.getElementById(`computers-${cabinetName}`);
    computerList.innerHTML = '';
    computers.forEach(computer => {
        const computerDiv = document.createElement('div');
        computerDiv.className = 'computer-item';
        computerDiv.setAttribute('data-name', computer.name); // Добавляем атрибут data-name

        const computerButton = document.createElement('button');
        computerButton.className = 'to_computer';
        computerButton.textContent = computer.name;
        computerButton.onclick = () => showCharacteristicsForm(cabinetName, computer.name);

        const deleteButton = document.createElement('button');
        deleteButton.className = 'delete-computer-btn';
        deleteButton.textContent = 'x';
        deleteButton.onclick = () => deleteComputer(cabinetName, computer.name);
        computerDiv.appendChild(computerButton);
        computerDiv.appendChild(deleteButton);
        computerList.appendChild(computerDiv);
    });
}

async function fetchCabinets() {
    const response = await fetch('http://127.0.0.1:5001/getCabinets');
    const cabs = await response.json();
    return cabs;  
}

// Показ формы для ввода характеристик с уже существующими значениями
async function showCharacteristicsForm(cabinetName, computerName) {
    const contentDiv = document.querySelector('.mainGuidePage');
    contentDiv.innerHTML = `<h2>Характеристики для ${computerName}</h2>
        <form id="characteristics-form">
            <div id="characteristics-container"></div>
            <button type="submit" class="save_characterist">Сохранить характеристики</button>
        </form>`;

    const response = await fetch(`http://127.0.0.1:5001/get_characteristics/${cabinetName}/${computerName}`);
    const characteristics = await response.json();

    const container = document.getElementById('characteristics-container');
    container.innerHTML = '';

    characteristics.forEach(characteristic => {
        const characteristicsDiv = document.createElement('div');
        characteristicsDiv.className = 'each_characteristics';
        const label = document.createElement('label');
        label.className = 'characteristics-name';
        label.textContent = characteristic.name;

        const input = document.createElement('input');
        input.className = 'characteristics-input';
        input.type = 'text';
        input.name = characteristic.name;
        input.value = characteristic.value;

        characteristicsDiv.appendChild(label);
        characteristicsDiv.appendChild(document.createElement('br'))
        characteristicsDiv.appendChild(input);
        characteristicsDiv.appendChild(document.createElement('br'));
        container.appendChild(characteristicsDiv);
    });

    const form = document.getElementById('characteristics-form');
    form.addEventListener('submit', async (event) => {
        event.preventDefault();
        const formData = new FormData(form);
        const data = {};
        formData.forEach((value, key) => {
            data[key] = value;
        });

        await fetch(`http://127.0.0.1:5001/save_characteristics/${cabinetName}/${computerName}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ characteristics: data })
        });

        alert('Характеристики сохранены!');
    });
}



// Удаление оборудования
async function deleteComputer(cabinetName, computerName) {
    if (!confirm(`Вы уверены, что хотите удалить оборудование "${computerName}"? Все связанные характеристики будут удалены.`)) {
        return;
    }

    const response = await fetch(`http://127.0.0.1:5001/delete_computer/${cabinetName}/${computerName}`, {
        method: 'DELETE'
    });

    if (response.ok) {
        alert(`Оборудование "${computerName}" было успешно удалено.`);

        // Находим и удаляем элемент оборудования из DOM через атрибут data-name
        const computerDiv = document.querySelector(`#computers-${cabinetName} .computer-item[data-name="${computerName}"]`);
        if (computerDiv) {
            computerDiv.remove();
        }
    } else {
        alert('Ошибка при удалении оборудования.');
    }
}


</script>


<style>

.save_characterist{
    background-color: #5e71dc;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: justify;
    text-decoration: none;
    /* display: inline-block; */
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    margin-top: 2px;
    transition: background-color 0.3s ease;
    max-width: 100%;
    overflow: hidden;
    min-width: 350px;
    /* word-wrap: break-word; */
    border: solid 1px blue;
}

.characteristics-name{
    font-family: Arial, Helvetica, sans-serif;
    font-size: 16pt;
    margin-bottom: 5px;
}

.characteristics-input{
    font-family: Arial, Helvetica, sans-serif;
    width: 500px;
    height: 30px;
    font-size: 12pt;
    margin-bottom: 5px;
}

.mainGuidePage{
    background-color: white;
    width: 150vh;
    padding: 20px;
}

.instruction_items{
    margin-left: 4%;
}

.instructions{
    margin-bottom: 1%;
}

h2{
    font-family: Arial, sans-serif;
    background-color: #ffffff;
    color: #434343;
    margin: 0px
}

h1{
    font-family: Arial, sans-serif;
    background-color: #ffffff;
    color: #434343;
    text-align: center;
    margin: 0px
}

.top_main_info,
.main_info{
    font-family: Arial, sans-serif;
    background-color: #ffffff;
    color: #333;
}
.sidebar {
    overflow: hidden;
    background-color: #5263c3;
    padding: 20px;
    width: 500px;
    color: white;
    border-right: 2px solid;
    border-color: #5555db;
    height: 79.5vh;
    overflow-y: auto;
}
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}
aside {
    display: block;
    unicode-bidi: isolate;
}

.eachCabinetButton{
    background-color: #3e4da2;
    margin-bottom: 10px;
    padding: 10px;
    display: block;
    justify-content: space-between;
    align-items: center;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
    color: white;
}

.to_computer{
    background-color: #5e71dc;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: center;
    text-decoration: none;
    /* display: inline-block; */
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    margin-right: 6px;
    transition: background-color 0.3s ease;
    max-width: 100%;
    overflow: hidden;
    width: 75%;
    /* min-width: 150px; */
    margin-top: 5px;
    margin-bottom: 5px;
}

.cabinetButton{
    background-color: #5e71dc;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: center;
    text-decoration: none;
    /* display: inline-block; */
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    margin-right: 6px;
    transition: background-color 0.3s ease;
    max-width: 100%;
    overflow: hidden;
    min-width: 150px;
    margin-top: 5px;
    margin-bottom: 5px;
}
.deleteCabinetButton,
.delete-computer-btn {
    background-color: rgb(254, 102, 102);
    color: white;
    border: none;
    border-radius: 50%;
    width: 24px;
    height: 24px;
    text-align: center;
    font-size: 14px;
    line-height: 24px;
    cursor: pointer;
    margin-left: 10px;
    margin-bottom: 5px;
}

.addItemInCabinetButton {
    background-color: #5e71dc;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    transition: background-color 0.3s ease;
    width: 210px;
}

.addItemInCabinetButton:hover,
.cabinetButton:hover,
.to_computer:hover,
.save_characterist:hover {
    background-color: #394462;
}

#addCabinetButton,
#toMainPage {
    background-color: #3e4da2;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    margin-top: 10px;
    transition: background-color 0.3s ease;
    margin-right: 5px;
}

#addCabinetButton:hover,
#toMainPage:hover {
    background-color: #394462;
}

.sidebar::-webkit-scrollbar-thumb {
    background-color: #4c59a2; /* Цвет ползунка */
    border-radius: 10px; /* Скругление углов ползунка */
    border: 3px solid #4c59a2; /* Обводка вокруг ползунка */
}


.sidebar::-webkit-scrollbar-track {
    background: #5e71dc; /* Цвет фона дорожки */
    border-radius: 10px; /* Скругление углов дорожки */
}

.sidebar::-webkit-scrollbar {
    width: 12px; /* Ширина скролл бара */
}

.add-computer-btn {
    background-color: #5e71dc;
    border: none;
    color: white;
    padding: 8px 12px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12pt;
    cursor: pointer;
    border-radius: 5px;
    margin-top: 5px;
    transition: background-color 0.3s ease;
    width: 210px;
}

#obor-name, #obor-type{
    font-family: Arial, Helvetica, sans-serif;
    font-size: 16pt;
}

#type, #name {
    width: 500px;
    height: 30px;
}
</style>