import React, { useState } from 'react';

// --- Your Updated Schema Definition ---
const schemaWithConditions = {
  contributionsAndDeferrals: {
    type: 'object',
    title: 'Contributions & Deferrals',
    description: 'Details of contributions made by the employer and employee.',
    'x-ui-order': [
      'employeeContributionsTypes',
      'participantContributionElectionOptions',
      'offerCatchupProvision',
      'payrollFrequency',
    ],
    properties: {
      employeeContributionsTypes: {
        type: 'object',
        title: 'Employee Contribution Types', // Section Title
        'x-ui-order': [
          'preTaxElectiveDeferrals',
          'preTaxRollover',
          'rothElectiveDeferrals',
          'rothRollover',
          'inPlanRothRollover',
          'inPlanRothTransfer',
          'inPlanRothTransferContributionTypes',
          'voluntaryAfterTaxEmployeeContributions',
        ],
        properties: {
          preTaxElectiveDeferrals: { type: 'string', title: 'Pre-Tax Elective Deferrals', element: 'checkbox' },
          preTaxRollover: { type: 'string', title: 'Pre-Tax Rollover', element: 'checkbox' },
          rothElectiveDeferrals: { type: 'string', title: 'Roth Elective Deferrals', element: 'checkbox' },
          rothRollover: { type: 'string', title: 'Roth Rollover', element: 'checkbox' },
          inPlanRothRollover: { type: 'string', title: 'In-Plan Roth Rollover', element: 'checkbox' /*, description: "The participant must meet a distributable event." // Original description, add back if needed */ },
          inPlanRothTransfer: { type: 'string', title: 'In-Plan Roth Transfer', element: 'checkbox' /*, description: "The participant doesn't have to meet a distributable event." // Original description, add back if needed */ },
          inPlanRothTransferContributionTypes: {
            type: 'object',
            title: 'Select all contributions types are eligible for in-plan Roth transfers', // Nested Object Title
            'x-ui-visible-if': { // Assuming you might want this conditional visibility from original schema
              "field": "contributionsAndDeferrals.employeeContributionsTypes.inPlanRothTransfer",
              "isNotEmpty": true
            },
            'x-ui-order': [
              'inPlanRothElectiveDeferrals', 'inPlanRothTransferRollover', 'inPlanRothEmployerMatchNonSafeHarbor',
              'inPlanRothSafeHarborMatch', 'inPlanRothSafeHarborNonElective', 'inPlanRothQacaMatch',
              'inPlanRothQacaNonElective', 'inPlanRothProfitSharing', 'inPlanRothQnec', 'inPlanRothQmac',
              'inPlanRothOtherEmployerContributionTypes', 'inPlanRothOtherEmployerContributionTypeName',
            ],
            properties: {
              inPlanRothElectiveDeferrals: { type: 'string', title: 'Elective deferrals', element: 'checkbox' },
              inPlanRothTransferRollover: { type: 'string', title: 'Rollover', element: 'checkbox' },
              inPlanRothEmployerMatchNonSafeHarbor: { type: 'string', title: 'Employer match (non-safe harbor)', element: 'checkbox' },
              inPlanRothSafeHarborMatch: { type: 'string', title: 'Safe harbor match', element: 'checkbox' },
              inPlanRothSafeHarborNonElective: { type: 'string', title: 'Safe harbor non-elective', element: 'checkbox' },
              inPlanRothQacaMatch: { type: 'string', title: 'Qualified automatic contributions arrangements (QACA) match', element: 'checkbox' },
              inPlanRothQacaNonElective: { type: 'string', title: 'Qualified automatic contributions arrangements (QACA) non elective', element: 'checkbox' },
              inPlanRothProfitSharing: { type: 'string', title: 'Profit sharing', element: 'checkbox' },
              inPlanRothQnec: { type: 'string', title: 'Qualified nonelective contribution (QNEC)', element: 'checkbox' },
              inPlanRothQmac: { type: 'string', title: 'Qualified matching contribution (QMAC)', element: 'checkbox' },
              inPlanRothOtherEmployerContributionTypes: { type: 'string', title: 'Other matching contribution type(s)', element: 'checkbox' },
              inPlanRothOtherEmployerContributionTypeName: { type: 'string', title: 'ENTER CONTRIBUTION TYPE', element: 'textbox' },
            },
          },
          voluntaryAfterTaxEmployeeContributions: { type: 'string', title: 'Voluntary employee after-tax', element: 'checkbox' },
        },
      },
      participantContributionElectionOptions: {
        type: 'radio', // Explicitly 'radio'
        title: 'Participant Contribution Election Options',
        enum: ['Percentage', 'FlatAmount', 'Both'],
        description: 'How participants can elect contributions.',
      },
      offerCatchupProvision: {
        type: 'boolean', // Will be rendered as a checkbox
        title: 'Offer Catch-up Provision (Age 50+)',
      },
      payrollFrequency: {
        type: 'string',
        title: 'Payroll Frequency',
        enum: ['Weekly', 'Bi-Weekly', 'Semi-Monthly', 'Monthly'],
        element: 'radio', // Explicitly 'radio'
      },
    },
  },
};

// --- Sample Data (Using the original sample data you provided earlier) ---
const sampleData = {
  "contributionsAndDeferrals": {
    "employeeContributionsTypes": {
      "preTaxElectiveDeferrals": "preTaxElectiveDeferrals",
      "preTaxRollover": "RolloverEnabled",
      "rothElectiveDeferrals": "Participant Allows Roth",
      "rothRollover": "", // Unchecked
      "inPlanRothRollover": "Enabled",
      "inPlanRothTransfer": "inPlanRothTransfer", // This will make inPlanRothTransferContributionTypes visible
      "inPlanRothTransferContributionTypes": { // Data for the nested object
        "inPlanRothElectiveDeferrals": "Selected",
        "inPlanRothTransferRollover": "",
        "inPlanRothEmployerMatchNonSafeHarbor": "Active",
        // Other new fields from schema will be missing data -> appear unchecked/empty
      },
      "voluntaryAfterTaxEmployeeContributions": "IsChecked" // Checked as per new schema (element: 'checkbox')
    },
    "participantContributionElectionOptions": "Both",
    "offerCatchupProvision": true,
    "payrollFrequency": "Bi-Weekly"
  }
};


// --- Helper Functions (UNCHANGED from previous response) ---
const getNestedValue = (obj, path, defaultValue = undefined) => {
  if (!path) return defaultValue;
  const properties = path.split('.');
  let current = obj;
  for (let i = 0; i < properties.length; i++) {
    if (current === null || typeof current !== 'object' || !current.hasOwnProperty(properties[i])) {
      return defaultValue;
    }
    current = current[properties[i]];
  }
  return current;
};

const evaluateCondition = (condition, fullData) => {
  if (!condition) return true;
  const { field, hasValue, isNotEmpty, isEmpty } = condition;
  let fieldValue = getNestedValue(fullData, field);
  if (typeof hasValue === 'boolean' && typeof fieldValue === 'string') {
    const lowerCaseData = fieldValue.toLowerCase().trim();
    fieldValue = !(lowerCaseData === 'false' || lowerCaseData === 'no' || lowerCaseData === '');
  }
  if (hasValue !== undefined) return fieldValue === hasValue;
  if (isNotEmpty !== undefined) {
    if (Array.isArray(fieldValue)) return fieldValue.length > 0 === isNotEmpty;
    if (typeof fieldValue === 'string') return (fieldValue.trim() !== '') === isNotEmpty;
    if (typeof fieldValue === 'boolean') return fieldValue === isNotEmpty;
    return (fieldValue !== undefined && fieldValue !== null) === isNotEmpty;
  }
  if (isEmpty !== undefined) {
    if (Array.isArray(fieldValue)) return fieldValue.length === 0 === isEmpty;
    if (typeof fieldValue === 'string') return (fieldValue.trim() === '') === isEmpty;
    if (typeof fieldValue === 'boolean') return !fieldValue === isEmpty;
    return (fieldValue === undefined || fieldValue === null) === isEmpty;
  }
  return true;
};

// --- UI Helper: InfoIcon (UNCHANGED from previous response) ---
const InfoIcon = ({ className = "w-4 h-4 text-gray-400 inline-block ml-1" }) => (
  <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" strokeWidth={1.5} stroke="currentColor" className={className}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z" />
  </svg>
);

// --- UI Component: SectionDisplay (UNCHANGED from previous response) ---
const SectionDisplay = ({ title, description, children }) => (
  <div className="bg-white p-6 md:p-8 mb-6 rounded-lg shadow">
    {title && (
      <h2 className="text-xl font-semibold text-gray-800 border-b border-gray-200 pb-3 mb-4">
        {title}
      </h2>
    )}
    {description && <p className="text-sm text-gray-600 mb-6">{description}</p>}
    <div className="space-y-5">
      {children}
    </div>
  </div>
);

// --- UI Component: CheckboxDisplay (UNCHANGED from previous response) ---
const CheckboxDisplay = ({ title, isChecked, description, dataValue }) => (
  <div className="flex items-start py-2">
    <div className="flex items-center h-5 mt-0.5">
      {isChecked ? (
        <div className="w-5 h-5 bg-blue-600 border-blue-600 rounded flex items-center justify-center">
          <svg className="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="3" d="M5 13l4 4L19 7"></path>
          </svg>
        </div>
      ) : (
        <div className="w-5 h-5 border-2 border-gray-400 rounded"></div>
      )}
    </div>
    <div className="ml-3 text-sm">
      <label htmlFor={title} className="font-medium text-gray-700 cursor-default">
        {title}
      </label>
      {description && (
        <span className="text-gray-500 text-xs ml-1">
           ({description}
          <span title={description}><InfoIcon className="w-3 h-3 text-gray-400 inline-block ml-0.5 relative -top-px" /></span>)
        </span>
      )}
    </div>
  </div>
);

// --- UI Component: RadioGroupDisplay (UNCHANGED from previous response) ---
const RadioGroupDisplay = ({ title, description, options, selectedValue, namePrefix }) => (
  <fieldset className="py-2">
    <legend className="text-sm font-semibold text-gray-800">{title}</legend>
    {description && <p className="text-xs text-gray-600 mt-1 mb-2">{description}</p>}
    <div className="mt-2 space-y-2">
      {options.map((option) => (
        <div key={`${namePrefix}-${option}`} className="flex items-center"> {/* Ensured key is unique with namePrefix */}
          <div className="flex items-center h-5">
            {selectedValue === option ? (
              <div className="w-4 h-4 border-2 border-blue-600 rounded-full flex items-center justify-center bg-blue-600">
                <div className="w-1.5 h-1.5 bg-white rounded-full"></div>
              </div>
            ) : (
              <div className="w-4 h-4 border-2 border-gray-400 rounded-full"></div>
            )}
          </div>
          <div className="ml-2 text-sm">
            <label htmlFor={`${namePrefix}-${option}`} className="font-medium text-gray-700 cursor-default">
              {option}
            </label>
          </div>
        </div>
      ))}
    </div>
  </fieldset>
);

// --- UI Component: FieldDisplay (UNCHANGED from previous response) ---
const FieldDisplay = ({ title, value, description }) => (
  <div className="py-2">
    <dt className="text-sm font-medium text-gray-700">
      {title}
      {description && (
        <span className="text-gray-500 text-xs ml-1">
          ({description}
          <span title={description}><InfoIcon className="w-3 h-3 text-gray-400 inline-block ml-0.5 relative -top-px" /></span>)
        </span>
      )}
    </dt>
    <dd className="mt-1 text-sm text-gray-800">{String(value)}</dd> {/* Ensure value is stringified */}
  </div>
);


// --- Core SchemaField Component (Dispatcher - ADJUSTED for new radio logic) ---
const SchemaField = ({ schemaNode, dataNode, fullData, renderMode, pathPrefix = '', isNested = false }) => {
  if (!schemaNode) return null;

  // Conditional Rendering Logic
  if (renderMode === 'conditional' && schemaNode['x-ui-visible-if']) {
    if (!evaluateCondition(schemaNode['x-ui-visible-if'], fullData)) {
      return null;
    }
  }

  const { type, title, properties, 'x-ui-order': order, description, element, enum: enumOptions } = schemaNode;
  const currentPath = pathPrefix.slice(0, -1); // Used for unique keys for radio names/field IDs

  // Object Type (Sections or Sub-sections)
  if (type === 'object' && properties) {
    const fieldKeys = order || Object.keys(properties);
    const content = (
      <div className={isNested ? "mt-4 pt-4 pl-4 border-l-2 border-gray-100" : ""}>
        {isNested && title && ( // Display title for nested objects if present
             <h3 className="text-md font-semibold text-gray-700 mb-3">{title}</h3>
        )}
        {/* Removed description for nested object title, as it's usually part of the parent section or field */}
        <div className="space-y-3">
          {fieldKeys.map(key => {
            const propertySchema = properties[key];
            if (!propertySchema) return null;
            const nestedData = dataNode && typeof dataNode === 'object' ? dataNode[key] : undefined;
            return (
              <SchemaField
                key={key}
                schemaNode={propertySchema}
                dataNode={nestedData}
                fullData={fullData}
                renderMode={renderMode}
                pathPrefix={`${pathPrefix}${key}.`}
                isNested={true}
              />
            );
          })}
        </div>
      </div>
    );

    if (!isNested) { // Top-level objects are main sections
      return <SectionDisplay title={title} description={description}>{content}</SectionDisplay>;
    }
    return content; // Nested objects are rendered inline
  }

  // Radio Button Type (handles element: 'radio' or type: 'radio' with enums)
  if ((element === 'radio' || type === 'radio') && enumOptions && Array.isArray(enumOptions)) {
    return (
      <RadioGroupDisplay
        title={title || 'Untitled Selection'}
        description={description}
        options={enumOptions}
        selectedValue={dataNode !== undefined && dataNode !== null ? String(dataNode) : undefined}
        namePrefix={currentPath}
      />
    );
  }

  // Checkbox Type
  if (element === 'checkbox' || type === 'boolean') {
    let isChecked = false;
    if (dataNode !== undefined && dataNode !== null) {
      isChecked = type === 'boolean' ? Boolean(dataNode) : (typeof dataNode === 'string' && dataNode.trim() !== '');
    }
    return (
      <CheckboxDisplay
        title={title || 'Untitled Checkbox'}
        isChecked={isChecked}
        description={description}
        dataValue={dataNode}
      />
    );
  }

  // Textbox or other simple string/number types (Default Fallback)
  let displayValue = <span className="italic text-gray-500">[N/A]</span>; // Italic for N/A
  if (dataNode !== undefined && dataNode !== null && String(dataNode).trim() !== '') {
    displayValue = String(dataNode);
  } else if (element === 'textbox' && (dataNode === undefined || String(dataNode).trim() === '')) {
     displayValue = <span className="italic text-gray-500">[Not specified]</span>; // More specific for empty textboxes
  }


  return (
    <FieldDisplay
      title={title || 'Untitled Field'}
      value={displayValue}
      description={description} // Pass description, FieldDisplay will decide how to show it
    />
  );
};


// --- App Component (UNCHANGED from previous response) ---
// --- App Component (Corrected) ---
function App() {
  const [renderMode, setRenderMode] = useState('conditional');
  const [currentSchema] = useState(schemaWithConditions);
  const [currentData] = useState(sampleData);

  return (
    <div className="min-h-screen bg-slate-50 py-8 px-4 sm:px-6 lg:px-8 font-sans">
      <div className="container mx-auto max-w-3xl">
        <header className="mb-8 text-left">
          <h1 className="text-2xl font-semibold text-gray-900">
            {currentSchema.contributionsAndDeferrals?.title || "Plan Configuration"}
          </h1>
        </header>
        <main>
          {Object.keys(currentSchema).map(schemaKey => (
            <SchemaField
              key={schemaKey}
              schemaNode={currentSchema[schemaKey]}
              dataNode={currentData && currentData[schemaKey]}
              fullData={currentData}
              renderMode={renderMode}
              pathPrefix={`${schemaKey}.`}
              isNested={false}
            />
          ))}
        </main>
      </div>
    </div>
  );
}

export default App;
