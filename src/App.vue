<template>
  <div @click="activeDropdown = null">
    <nav>
      <img class="logo" src="/src/assets/favicon.ico" alt="Logo">
      <div class="nav_content">
        <a href="#">Fichas</a>
        <a href="#">Homebrew</a>
      </div>
    </nav>

    <div class="char_header">
      <label>Nome:</label>
      <input size="14" v-model="charName">

      <label>Classe:</label>
      <select v-model="charClass">
        <option value="Combatente">Combatente</option>
        <option value="Especialista">Especialista</option>
        <option value="Ocultista">Ocultista</option>
      </select>

      <label>Nível:</label>
      <select v-model="charLvl">
        <option v-for="i in 20" :key="i" :value="i">{{ i }}</option>
      </select>

      <label>NEX:</label>
      <input size="3" v-model="charNex">
    </div>

    <div class="char_sheet">
      <section>
        <div class="attributes">
          <img src="/src/assets/attributes.png" alt="Atributos">
          <input id="agi" type="number" v-model.number="attributes.agi">
          <input id="str" type="number" v-model.number="attributes.str">
          <input id="int" type="number" v-model.number="attributes.int">
          <input id="pre" type="number" v-model.number="attributes.pre">
          <input id="vig" type="number" v-model.number="attributes.vig">
        </div>
        <div class="stats"> 
          <section class="desloc-n-turn-pe">
            <div>
              <input type="number" v-model.number="turnPE"><br>
              <label>PE/Turno</label>
            </div>
            <div>
              <input type="number" v-model.number="metersDesloc"> <label>m /</label>
              <input size="2" type="number" v-model.number="squareDesloc"> <label>q</label><br>
              <label>Deslocamento</label>
            </div>
          </section>

          <section class="status-bars">
          <label>PONTOS DE VIDA</label><br>
            <div class="bar-container">
              <div class="bar-fill pv-fill" :style="{ width: pvPercentage + '%' }"></div>
              <div class="bar-content">
                <input type="number" v-model.number="currentPV"> 
                <span>/</span> 
                <input type="number" v-model.number="totalPV">
              </div>
            </div>
            <br>
            <label>SANIDADE</label><br>
              <div class="bar-container">
                <div class="bar-fill san-fill" :style="{ width: sanPercentage + '%' }"></div>
                <div class="bar-content">
                  <input type="number" v-model.number="currentSAN"> 
                  <span>/</span> 
                  <input type="number" v-model.number="totalSAN">
                </div>
              </div>
              <br>
              <label>PONTOS DE ESFORÇO</label><br>
              <div class="bar-container">
                <div class="bar-fill pe-fill" :style="{ width: pePercentage + '%' }"></div>
                <div class="bar-content">
                  <input type="number" v-model.number="currentPE"> 
                  <span>/</span> 
                  <input type="number" v-model.number="totalPE">
                </div>
              </div>
              <br>
          </section>

          <section class="defenses">
            <label>Defesa:</label> 
            <span id="total_defense">{{ totalDefense }}</span> = 10 + AGI + 
            <input size="4" placeholder="Equip." type="number" v-model.number="equipDefense">&nbsp;
            <input size="4" placeholder="Outros" type="number" v-model.number="otherDefense">
          </section>

          <section class="dodge">
            <label>Esquiva:</label>
            <span id="dodge">{{ dodgeValue }}</span>
          </section>

          <section class="block">
            <label>Bloqueio:</label>
            <span id="block">{{ blockValue }}</span>
          </section>

          <section class="proficiency">
            <div><label>Proteção:</label><input v-model="proficiencies.protection"></div>
            <div><label>Resistências:</label><input v-model="proficiencies.resistances"></div>
            <div><label>Proficiências:</label><input v-model="proficiencies.skills"></div>
          </section>
        </div>
      </section>

      <div class="skills">
        <table>
          <thead>
            <tr>
              <th>Perícia</th>
              <th>Dados</th>
              <th>Treino</th>
              <th>Bônus</th>
              <th></th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(skill, index) in skills" :key="skill.name">
              <td>{{ skill.name }}</td>
              <td>
                <div class="dropdown">
                  <button 
                    class="dropdown_button attrSelect" 
                    @click.stop="toggleDropdown('skill_attr_' + skill.name)">
                    {{ getAttributeLabel(skill.attribute) }}
                  </button>
                  <div
                    :class="['dropdown_content', { 'dropdown_up': index >= 22 }]" 
                    v-show="activeDropdown === 'skill_attr_' + skill.name">
                    <div 
                      class="dropdown_item" 
                      v-for="attr in skillAttributes" 
                      :key="attr.value" 
                      @click="selectSkillDropdown(skill, attr.value)">
                      {{ attr.label }}
                    </div>
                  </div>
                </div>
              </td>
              <td>
                <select class="skillLvl" v-model.number="skill.train">
                  <option :value="0">0</option>
                  <option :value="5">5</option>
                  <option :value="10">10</option>
                  <option :value="15">15</option>
                </select>
              </td>
              <td><input size="2" type="number" class="skill" v-model.number="skill.bonus"></td>
              <td><span>=</span></td>
              <td><span class="sum">{{ getSkillTotal(skill) }}</span></td>
            </tr>
          </tbody>
        </table>
        <i class="skills_notes">+ Penalidade de Carga. * Somente Treinado.</i>
      </div>

      <div class="ability">
        <div class="buttons_tab">
          <button 
            v-for="tab in tabs" :key="tab.id"
            :class="['tab_btn', { active: activeTab === tab.id }]" 
            @click="activeTab = tab.id"
          >
            {{ tab.label }}
          </button>
        </div>
        <button v-if="activeTab !== 'notas'" class="add_hab" @click="openModal">Adicionar</button>
        <br>

        <div class="tab_content active">
          <div v-if="activeTab === 'notas'" class="notes_container">
            <textarea 
              v-model="notesPersonal" 
              class="notes_textarea" 
              placeholder="Anotações pessoais."
            ></textarea>
            <textarea 
              v-model="notesAppearance" 
              class="notes_textarea" 
              placeholder="Aparência, nome, gênero, personalidade, etc."
            ></textarea>
            <textarea 
              v-model="notesHistory" 
              class="notes_textarea" 
              placeholder="Histórico, relações e objetivos."
            ></textarea>
          </div>
          <div v-else v-for="(ab, index) in filteredAbilities" :key="index" class="ability_card">
            <div class="added_ability">
              <div>
                <b class="title">{{ ab.name }}</b>
                <p v-if="ab.element" :class="['circle_tag', ab.element.toLowerCase()]">
                  {{ ab.element.toUpperCase() }} {{ ab.circle[0] }}
                </p>
                <p v-if="activeTab === 'rituais'"><b>Execução:</b> {{ ab.execution }}</p>
                <p v-if="activeTab === 'rituais'"><b>Alcance:</b> {{ ab.range }}</p>
                <p v-if="ab.area"><b>Área:</b> {{ ab.area }}</p>
                <p v-if="ab.target"><b>Alvo:</b> {{ ab.target }}</p>
                <p v-if="ab.duration"><b>Duração:</b> {{ ab.duration }}</p>
                <p v-if="ab.effect"><b>Efeito:</b> {{ ab.effect }}</p>
                <p v-if="ab.resists"><b>Resistências:</b> {{ ab.resists }}</p>

                <p v-if="ab.damage"><b>Dano:</b> {{ ab.damage }}</p>
                <p v-if="ab.critical"><b>Crítico:</b> {{ ab.critical }}</p>
                <p v-if="ab.multiplier"><b>Multiplicador:</b> {{ ab.multiplier }}</p>
                <p v-if="ab.bonus"><b>Ataque Bônus:</b> {{ ab.bonus }}</p>
                <p v-if="ab.type"><b>Tipo de Dano:</b> {{ ab.type }}</p>
                <p v-if="ab.weapon_range"><b>Alcance:</b> {{ ab.weapon_range }}</p>
                <p v-if="ab.skill"><b>Perícia:</b> {{ ab.skill }}</p>
                <p v-if="ab.damage_attribute"><b>Atributo de Dano:</b> {{ ab.damage_attribute }}</p>

                <p v-if="ab.category"><b>Categoria:</b> {{ ab.category }}</p>
                <p v-if="ab.weight"><b>Peso:</b> {{ ab.weight }}</p>
              </div>
              <div class="text ql-editor" v-html="ab.content"></div>
              <button class="delete_btn" @click="removeAbility(ab)">REMOVER</button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="isModalOpen" class="modal" id="modal" @click.self="closeModal">
      <div class="modal_content">

        <template v-if="activeTab === 'rituais'">
          <h3>Novo Ritual</h3>
          <div class="modal_row_top">
            <section>
              <label>Nome*</label><br>
              <input v-model="newAbility.name" class="dark_input" id="ability_name">
            </section>
            
            <div class="dropdown">
              <label>Elemento</label>
              <button class="dropdown_button" id="btn_element" @click.stop="toggleDropdown('element')">{{ newAbility.element }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'element'">
                <div class="dropdown_item" v-for="el in dropdownOptions.elements" :key="el" @click="selectDropdown('element', el)">{{ el }}</div>
              </div>
            </div>

            <div class="dropdown">
              <label>Círculo</label>
              <button class="dropdown_button" id="btn_circle" @click.stop="toggleDropdown('circle')">{{ newAbility.circle }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'circle'">
                <div class="dropdown_item" v-for="circ in dropdownOptions.circles" :key="circ" @click="selectDropdown('circle', circ)">{{ circ }}</div>
              </div>
            </div>

            <div class="dropdown">
              <label>Execução</label>
              <button class="dropdown_button" id="btn_execution" @click.stop="toggleDropdown('execution')">{{ newAbility.execution }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'execution'">
                <div class="dropdown_item" v-for="ex in dropdownOptions.executions" :key="ex" @click="selectDropdown('execution', ex)">{{ ex }}</div>
              </div>
            </div>
          </div>

          <div class="flex_row">
            <div class="dropdown">
              <label>Alcance</label>
              <button class="dropdown_button" id="btn_range" @click.stop="toggleDropdown('range')">{{ newAbility.range }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'range'">
                <div class="dropdown_item" v-for="rg in dropdownOptions.ranges" :key="rg" @click="selectDropdown('range', rg)">{{ rg }}</div>
              </div>
            </div>
            <div><label>Área</label><br><input v-model="newAbility.area" class="dark_input"></div>
            <div><label>Alvo</label><br><input v-model="newAbility.target" class="dark_input"></div>
            <div><label>Duração</label><br><input v-model="newAbility.duration" class="dark_input"></div>
          </div>

          <div class="flex_row">
            <div><label>Efeito</label><br><input v-model="newAbility.effect" class="dark_input"></div>
            <div><label>Resistências</label><br><input v-model="newAbility.resists" class="dark_input"></div>
          </div>

          <div class="flex_row">
            <div><label>Dados Normal</label><br><input v-model="newAbility.normal_dice" class="dark_input"></div>
            <div><label>Dados Discente</label><br><input v-model="newAbility.discent_dice" class="dark_input"></div>
            <div><label>Dados Verdadeiro</label><br><input v-model="newAbility.true_dice" class="dark_input"></div>
          </div>
        </template>
        
        <template v-else-if="activeTab === 'combate'">
          <h3>Novo Ataque</h3>
          
          <div class="modal_row_top">
            <div><label>Nome*</label><br><input v-model="newAbility.name" class="dark_input"></div>
            <div><label>Dano*</label><br><input v-model="newAbility.damage" class="dark_input"></div>
            <div><label>Crítico*</label><br><input v-model="newAbility.critical" class="dark_input"></div>
            <div><label>Multiplicador*</label><br><input v-model="newAbility.multiplier" class="dark_input"></div>
          </div>

          <div class="flex_row">
            <div><label>Ataque Bônus</label><br><input v-model="newAbility.bonus" class="dark_input"></div>
            <div><label>Tipo de Dano</label><br><input v-model="newAbility.type" class="dark_input"></div>
          </div>

          <div class="flex_row">
            <div class="dropdown">
              <label>Alcance</label>
              <button class="dropdown_button" id="btn_weapon_range" @click.stop="toggleDropdown('weapon_range')">
                {{ newAbility.weapon_range || '-' }}
              </button>
              <div class="dropdown_content" v-show="activeDropdown === 'weapon_range'">
                <div class="dropdown_item" v-for="da in dropdownOptions.weapon_range" :key="da" @click="selectDropdown('weapon_range', da)">
                  {{ da }}
                </div>
              </div>
            </div>

            <div class="dropdown">
              <label>Perícia</label>
              <input v-model="newAbility.skill" class="dark_input">
            </div>

            <div class="dropdown">
              <label>Atributo</label>
              <button class="dropdown_button" id="btn_damage" @click.stop="toggleDropdown('damage_attribute')">{{ newAbility.damage_attribute || 'Força' }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'damage_attribute'">
                <div class="dropdown_item" v-for="da in dropdownOptions.damage_attribute" :key="da" @click="selectDropdown('damage_attribute', da)">{{ da }}</div>
              </div>
            </div>

          </div>
        </template>

        <template v-else-if="activeTab === 'habilidade'">
          <h3>Nova Habilidade</h3>
          
          <div class="modal_row_top">
            <div><label>Nome*</label><br><input v-model="newAbility.name" class="dark_input"></div>
          </div>

        </template>

        <template v-else-if="activeTab === 'inventario'">
          <h3>Novo Item</h3>
          
          <div class="modal_row_top">
            <div><label>Nome*</label><br><input v-model="newAbility.name" class="dark_input"></div>
            <div class="dropdown">
              <label>Categoria*</label>
              <button class="dropdown_button" id="btn_cat" @click.stop="toggleDropdown('category')">{{ newAbility.category || '0' }}</button>
              <div class="dropdown_content" v-show="activeDropdown === 'category'">
                <div class="dropdown_item" v-for="cat in dropdownOptions.category" :key="cat" @click="selectDropdown('category', cat)">{{ cat }}</div>
              </div>
            </div>
            <div><label>Peso*</label><br><input v-model="newAbility.weight" class="dark_input"></div>
          </div>

        </template>
        <section>
          <label>Descrição*</label>
          <div ref="editorRef" id="text_editor"></div>
        </section>

        <div class="modal_actions">
          <br>
          <button id="save_ability" @click="saveAbility">Salvar</button>
          <button id="cancel" @click="closeModal">Cancelar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, nextTick, effect } from 'vue';
import Quill from 'quill';
import 'quill/dist/quill.snow.css';

// ESTADOS GERAIS
const charName = ref('');
const charClass = ref('Combatente');
const charLvl = ref(1);
const charNex = ref('0%');

const attributes = reactive({ agi: 1, str: 1, int: 1, pre: 1, vig: 1 });
const proficiencies = reactive({ protection: '', resistances: '', skills: '' });

const equipDefense = ref(null);
const otherDefense = ref(null);
const metersDesloc = ref(9);

const squareDesloc = computed({
  get: () => metersDesloc.value / 1.5,
  set: (val) => metersDesloc.value = val * 1.5
});

const notesPersonal = ref('');
const notesAppearance = ref('');
const notesHistory = ref('');

watch(equipDefense, (val) => {
  if (val > 99) equipDefense.value = 99;
  if (val < -99) equipDefense.value = -99;
});

watch(otherDefense, (val) => {
  if (val > 99) otherDefense.value = 99;
  if (val < -99) otherDefense.value = -99;
});

// LÓGICA DE CLASSE
const classInfo = {
  Combatente: { basePV: 20, baseSAN: 12, basePE: 2, lvlPV: 4, lvlSAN: 3, lvlPE: 2 },
  Especialista: { basePV: 16, baseSAN: 16, basePE: 3, lvlPV: 3, lvlSAN: 4, lvlPE: 3 },
  Ocultista: { basePV: 12, baseSAN: 20, basePE: 4, lvlPV: 2, lvlSAN: 5, lvlPE: 4 }
};

// Valores base calculados automaticamente
const basePV = computed(() => classInfo[charClass.value].basePV + (classInfo[charClass.value].lvlPV * (charLvl.value - 1)) + (attributes.vig) * charLvl.value);
const baseSAN = computed(() => classInfo[charClass.value].baseSAN + (classInfo[charClass.value].lvlSAN * (charLvl.value - 1)));
const basePE = computed(() => classInfo[charClass.value].basePE + (classInfo[charClass.value].lvlPE * (charLvl.value - 1)) + (attributes.pre) * charLvl.value);

// Bônus modificáveis inseridos pelo usuário
const pvBonus = ref(0);
const sanBonus = ref(0);
const peBonus = ref(0);

// Totais com Getter e Setter (permitem modificação na tela)
const totalPV = computed({
  get: () => basePV.value + pvBonus.value,
  set: (val) => pvBonus.value = val - basePV.value
});

const totalSAN = computed({
  get: () => baseSAN.value + sanBonus.value,
  set: (val) => sanBonus.value = val - baseSAN.value
});

const totalPE = computed({
  get: () => basePE.value + peBonus.value,
  set: (val) => peBonus.value = val - basePE.value
});

// Suas variáveis "current" e os "watchers" continuam iguais logo abaixo daqui...
const currentPV = ref(totalPV.value);
const currentSAN = ref(totalSAN.value);
const currentPE = ref(totalPE.value);

watch(totalPV, (newVal, oldVal) => { if (currentPV.value === oldVal || currentPV.value === 0) currentPV.value = newVal; }, { immediate: true });
watch(totalSAN, (newVal, oldVal) => { if (currentSAN.value === oldVal || currentSAN.value === 0) currentSAN.value = newVal; }, { immediate: true });
watch(totalPE, (newVal, oldVal) => { if (currentPE.value === oldVal || currentPE.value === 0) currentPE.value = newVal; }, { immediate: true });

const turnPeBonus = ref(0);
const turnPE = computed({
  get: () => charLvl.value + turnPeBonus.value,
  set: (val) => turnPeBonus.value = val - charLvl.value
});

// PERÍCIAS

const skillAttributes = [
  { label: 'AGI', value: 'agi' },
  { label: 'FOR', value: 'str' },
  { label: 'INT', value: 'int' },
  { label: 'PRE', value: 'pre' },
  { label: 'VIG', value: 'vig' }
];

const getAttributeLabel = (val) => {
  const attr = skillAttributes.find(a => a.value === val);
  return attr ? attr.label : val.toUpperCase();
};

const selectSkillDropdown = (skill, value) => {
  skill.attribute = value;
  activeDropdown.value = null; 
};

const rawSkills = [
  {name: "Acrobacia+", attribute: "agi"}, {name: "Adestramento*", attribute: "pre"}, {name: "Artes*", attribute: "pre"},
  {name: "Atletismo", attribute: "str"}, {name: "Atualidades", attribute: "int"}, {name: "Ciências*", attribute: "int"},
  {name: "Crime*+", attribute: "agi"}, {name: "Diplomacia", attribute: "pre"}, {name: "Enganação", attribute: "pre"},
  {name: "Fortitude", attribute: "vig"}, {name: "Furtividade+", attribute: "agi"}, {name: "Iniciativa", attribute: "agi"},
  {name: "Intimidação", attribute: "pre"}, {name: "Intuição", attribute: "pre"}, {name: "Investigação", attribute: "int"},
  {name: "Luta", attribute: "str"}, {name: "Medicina", attribute: "int"}, {name: "Ocultismo*", attribute: "int"},
  {name: "Percepção", attribute: "pre"}, {name: "Pilotagem*", attribute: "agi"}, {name: "Pontaria", attribute: "agi"},
  {name: "Profissão*", attribute: "int"}, {name: "Reflexos", attribute: "agi"}, {name: "Religião*", attribute: "pre"},
  {name: "Sobrevivência", attribute: "int"}, {name: "Tática*", attribute: "int"}, {name: "Tecnologia*", attribute: "int"},
  {name: "Vontade", attribute: "pre"}
];

const skills = reactive(rawSkills.map(s => ({ ...s, train: 0, bonus: 0 })));

const getSkillTotal = (skill) => Math.min(Math.max((skill.train || 0) + (skill.bonus || 0), -99), 114);
const getSkillValueByName = (name) => {
  const skill = skills.find(s => s.name === name);
  return skill ? getSkillTotal(skill) : 0;
};

watch(skills, (newSkills) => {
  newSkills.forEach(skill => {
    if (skill.bonus > 99) skill.bonus = 99;
    if (skill.bonus < -99) skill.bonus = -99;
    
    if (skill.train === undefined) skill.train = 0;
  });
}, { deep: true });

// DEFESAS
const totalDefense = computed(() => 10 + (attributes.agi || 0) + (equipDefense.value || 0) + (otherDefense.value || 0));
const dodgeValue = computed(() => totalDefense.value + getSkillValueByName("Reflexos"));
const blockValue = computed(() => getSkillValueByName("Fortitude"));

// HABILIDADES & MODAL
const tabs = [
  { id: 'combate', label: 'Combate' }, { id: 'habilidade', label: 'Habilidades' },
  { id: 'rituais', label: 'Rituais' }, { id: 'inventario', label: 'Inventário' }, { id: 'notas', label: 'Notas' }
];
const activeTab = ref('combate');
const abilities = ref([]);

const filteredAbilities = computed(() => abilities.value.filter(a => a.tab === activeTab.value));
const removeAbility = (abilityToRemove) => abilities.value = abilities.value.filter(a => a !== abilityToRemove);

const isModalOpen = ref(false);
const editorRef = ref(null);
let quillInstance = null;

const activeDropdown = ref(null);
const dropdownOptions = {
  elements: ['Conhecimento', 'Energia', 'Medo', 'Morte', 'Sangue', 'Varia'],
  circles: ['1º Círculo', '2º Círculo', '3º Círculo', '4º Círculo'],
  executions: ['Padrão', 'Movimento', 'Completa', 'Reação', 'Livre'],
  ranges: ['Pessoal', 'Toque', 'Curto', 'Médio', 'Longo', 'Extremo', 'Ilimitado'],
  damage_attribute: ['Agilidade', 'Força', 'Intelecto', 'Presença', 'Vigor'],
  weapon_range: ['-', 'Curto', 'Médio', 'Longo', 'Extremo', 'Ilimitado'],
  category: ['0', 'I', 'II', 'III', 'IV']
};

const newAbility = reactive({
  name: '', element: 'Conhecimento', circle: '1º Círculo', execution: 'Padrão',
  range: 'Pessoal', area: '', target: '', duration: '', effect: '', resists: '',
  normal_dice: '', discent_dice: '', true_dice: '',

  damage: '', critical: '', multiplier:'', bonus: '', type: '', skill: '', damage_attribute: '',

  category: '', weight: ''
});

const toggleDropdown = (menu) => { activeDropdown.value = activeDropdown.value === menu ? null : menu; };
const selectDropdown = (menu, value) => { newAbility[menu] = value; activeDropdown.value = null; };

const openModal = async () => {
  document.body.classList.add("no_scroll");
  isModalOpen.value = true;
  await nextTick();
  if (!quillInstance && editorRef.value) {
    quillInstance = new Quill(editorRef.value, {
      theme: 'snow',
      modules: { toolbar: [["bold", "italic", "underline"], [{ list: "ordered" }, { list: "bullet" }], ["link"], ["clean"]] }
    });
  }
};

const closeModal = () => {
  document.body.classList.remove("no_scroll");
  isModalOpen.value = false;
  newAbility.name = '';
  quillInstance = null;
  activeDropdown.value = null;
};

const saveAbility = () => {

  const content = quillInstance.getText().trim().length === 0 ? "<p>Sem Descrição</p>" : quillInstance.root.innerHTML;

  const abilityToSave = {
    tab: activeTab.value,
    name: newAbility.name.trim() || 'Nome Vazio',
    content: quillInstance.root.innerHTML, content
  }

  switch (activeTab.value) {
    case 'combate':
      Object.assign(abilityToSave, {
        damage: newAbility.damage,
        critical: newAbility.critical,
        multiplier: newAbility.multiplier,
        bonus: newAbility.bonus,
        type: newAbility.type,
        weapon_range: newAbility.weapon_range || '-',
        skill: newAbility.skill,
        damage_attribute: newAbility.damage_attribute || 'Força'
      })
      break;
    case 'rituais':
      Object.assign(abilityToSave, {
        element: newAbility.element, 
        circle: newAbility.circle, 
        execution: newAbility.execution, 
        range: newAbility.range, 
        area: newAbility.area, 
        target: newAbility.target, 
        duration: newAbility.duration, 
        effect: newAbility.effect, 
        resists: newAbility.resists
      })
      break;
    case 'habilidade':
      break;
    case 'inventario':
      Object.assign(abilityToSave, {
        category: newAbility.category || '0',
        weight: newAbility.weight || '0'
      })
    default:
      break;
  }

  abilities.value.push(abilityToSave);
  closeModal();
};

// BARRA DE PV, SANIDADE E PE
const pvPercentage = computed(() => {
  if (totalPV.value === 0) return 0;
  const percent = (currentPV.value / totalPV.value) * 100;
  return Math.max(0, Math.min(100, percent)); 
});

const sanPercentage = computed(() => {
  if (totalSAN.value === 0) return 0;
  const percent = (currentSAN.value / totalSAN.value) * 100;
  return Math.max(0, Math.min(100, percent)); 
});

const pePercentage = computed(() => {
  if (totalPE.value === 0) return 0;
  const percent = (currentPE.value / totalPE.value) * 100;
  return Math.max(0, Math.min(100, percent)); 
});
</script>