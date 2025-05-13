import React, { useState, useEffect } from 'react';

// Tailwind CSS will be used for styling. Ensure it's available in your project.
// For a standalone example, you might include: <script src="https://cdn.tailwindcss.com"></script> in your HTML.

// --- Schema Definition (User Provided Simplified Schema) ---
const schemaWithConditions = {
  "contributionsAndDeferrals": {
    "type": "object",
    "title": "Contributions & Deferrals",
    "description": "Details of contributions made by the employer and employee.",
    "x-ui-order": [ // This order might need adjustment if some fields are removed from properties
      "employeeContributionsTypes",
      "participantContributionElectionOptions",
      "offerCatchupProvision",
      "payrollFrequency",
      // "employerContributions", // Removed from user's simplified schema properties
      // "employerMatchNonSafeHarbor", // Removed from user's simplified schema properties
      // "employerMatchSafeHarbor" // Removed from user's simplified schema properties
    ],
    "properties": {
      "employeeContributionsTypes": {
        "type": "object",
        "title": "Employee Contribution Types",
        "x-ui-order": [
          "preTaxElectiveDeferrals",
          "preTaxRollover",
          "rothElectiveDeferrals",
          "rothRollover",
          "inPlanRothRollover",
          "inPlanRothTransfer",
          "inPlanRothTransferContributionTypes",
          "voluntaryAfterTaxEmployeeContributions"
        ],
        "properties": {
          "preTaxElectiveDeferrals": { "type": "string", "title": "Pre-Tax Elective Deferrals", "element" : "checkbox" },
          "preTaxRollover": { "type": "string", "title": "Pre-Tax Rollover", "element" : "checkbox" },
          "rothElectiveDeferrals": { "type": "string", "title": "Roth Elective Deferrals", "element" : "checkbox" }, // Corrected from "string": "boolean"
          "rothRollover": { "type": "string", "title": "Roth Rollover", "element" : "checkbox" },
          "inPlanRothRollover": { "type": "string", "title": "In-Plan Roth Rollover", "element" : "checkbox" },
          "inPlanRothTransfer": { "type": "string", "title": "In-Plan Roth Transfer", "element" : "checkbox" },
          "inPlanRothTransferContributionTypes": {
            "type": "object",
            "title": "In-Plan Roth Transfer Types",
            // "element" : "checkbox", // 'element' at object level is unusual, applying to properties instead.
            "x-ui-visible-if": { 
              "field": "contributionsAndDeferrals.employeeContributionsTypes.inPlanRothTransfer",
              // For string type checkboxes, "hasValue": true would mean the string is literally 'true'.
              // We'll use "isNotEmpty": true if a non-empty string means "checked" for visibility.
              // Or, if the string value itself is the condition: "hasValue": "someValueThatMeansChecked"
              // For now, let's assume a non-empty string for inPlanRothTransfer makes this section visible.
              "isNotEmpty": true 
            },
            "properties": {
              "inPlanRothElectiveDeferrals": { "type": "string", "title": "IPRT - Elective Deferrals", "element" : "checkbox" },
              "inPlanRothTransferRollover": { "type": "string", "title": "IPRT - Rollover", "element" : "checkbox" },
              "inPlanRothEmployerMatchNonSafeHarbor": { "type": "string", "title": "IPRT - Employer Match Non-SH", "element" : "checkbox" },
            }
          },
          "voluntaryAfterTaxEmployeeContributions": { "type": "string", "title": "Voluntary After-Tax (if any)", "element": "textbox" }
        }
      },
      "participantContributionElectionOptions": {
        "type": "radio",
        "title": "Participant Contribution Election Options",
        "enum": ["Percentage", "FlatAmount", "Both"],
        "description": "How participants can elect contributions."
      },
      "offerCatchupProvision": {
        "type": "boolean", // This is a true boolean in schema
        "title": "Offer Catch-up Provision (Age 50+)"
        // "element": "radio" // if you want radio, data needs to support it, and rendering logic too
      },
      "payrollFrequency": {
        "type": "string",
        "title": "Payroll Frequency",
        "enum": ["Weekly", "Bi-Weekly", "Semi-Monthly", "Monthly"]
      }
      // Properties for employerContributions, employerMatchNonSafeHarbor, employerMatchSafeHarbor 
      // were removed from this simplified schema properties, so they won't render.
    }
  }
};

// --- Sample Data (Adjusted for simplified schema and checkbox string values) ---
const sampleData = {
  "contributionsAndDeferrals": {
    "employeeContributionsTypes": {
      "preTaxElectiveDeferrals": "preTaxElectiveDeferrals", // Non-empty string = checked
      "preTaxRollover": "",                               // Empty string = unchecked
      "rothElectiveDeferrals": "Participant Allows Roth",  // Non-empty string = checked
      "rothRollover": "",                                
      "inPlanRothRollover": "Enabled",                   
      "inPlanRothTransfer": "inPlanRothTransfer", // Empty string, so inPlanRothTransferContributionTypes will be hidden in conditional
      "inPlanRothTransferContributionTypes": { // This object will be hidden in conditional mode due to above
        "inPlanRothElectiveDeferrals": "", 
        "inPlanRothTransferRollover": "", 
        "inPlanRothEmployerMatchNonSafeHarbor": ""
      },
      "voluntaryAfterTaxEmployeeContributions": "Available up to 5%" 
    },
    "participantContributionElectionOptions": "Percentage",
    "offerCatchupProvision": true, // Actual boolean data
    "payrollFrequency": "Bi-Weekly"
  }
};


// --- Helper Functions ---
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

  // If the schema condition expects a boolean (hasValue: true/false),
  // and the actual data is a string, interpret the string for the condition.
  if (typeof hasValue === 'boolean' && typeof fieldValue === 'string') {
    const lowerCaseData = fieldValue.toLowerCase().trim();
    if (lowerCaseData === 'false' || lowerCaseData === 'no' || lowerCaseData === '') {
      fieldValue = false;
    } else {
      fieldValue = true; 
    }
  }
  
  if (hasValue !== undefined) {
    return fieldValue === hasValue;
  }

  // isNotEmpty / isEmpty for strings and arrays
  if (isNotEmpty !== undefined) {
    if (Array.isArray(fieldValue)) return fieldValue.length > 0 === isNotEmpty;
    if (typeof fieldValue === 'string') return (fieldValue.trim() !== '') === isNotEmpty;
    // For other types (like boolean data for a checkbox that uses isNotEmpty for visibility)
    // if isNotEmpty is true, we want it to be visible if fieldValue is true-ish (not false, null, undefined)
    // if isNotEmpty is false, we want it to be visible if fieldValue is false-ish
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


// --- React Components ---
const SchemaField = ({ schemaNode, dataNode, fullData, renderMode, pathPrefix = '' }) => {
  if (!schemaNode) return null;

  if (renderMode === 'conditional' && schemaNode['x-ui-visible-if']) {
    if (!evaluateCondition(schemaNode['x-ui-visible-if'], fullData)) {
      return null;
    }
  }
  
  const { type, title, properties, 'x-ui-order': order, description, element } = schemaNode;

  if (type === 'object' && properties) {
    const fieldKeys = order || Object.keys(properties);
    return (
      <div className="p-4 mb-4 border border-gray-300 rounded-lg shadow-sm bg-white">
        {title && <h3 className="text-xl font-semibold mb-3 text-indigo-700 border-b pb-2">{title}</h3>}
        {description && <p className="text-sm text-gray-500 mb-3">{description}</p>}
        <div className="space-y-3">
          {fieldKeys.map(key => {
            const propertySchema = properties[key];
            if (!propertySchema) return null; 
            
            if (renderMode === 'conditional' && propertySchema.type !== 'object' && propertySchema['x-ui-visible-if']) {
                if (!evaluateCondition(propertySchema['x-ui-visible-if'], fullData)) {
                    return null;
                }
            }

            const nestedData = dataNode && typeof dataNode === 'object' ? dataNode[key] : undefined;
            const nextPathPrefix = `${pathPrefix}${key}.`; 

            return (
              <SchemaField
                key={key}
                schemaNode={propertySchema}
                dataNode={nestedData}
                fullData={fullData}
                renderMode={renderMode}
                pathPrefix={nextPathPrefix}
              />
            );
          })}
        </div>
      </div>
    );
  }

  // Handle specific element types
  if (element === 'checkbox') {
    // For string type checkboxes, a non-empty string means "checked".
    // For boolean type checkboxes, true means "checked".
    let isChecked = false;
    if (dataNode !== undefined && dataNode !== null) {
        if (type === 'boolean') {
            isChecked = Boolean(dataNode);
        } else if (typeof dataNode === 'string') {
            isChecked = dataNode.trim() !== '';
        }
    }
    const checkboxIcon = isChecked ? '☑' : '☐';
    return (
      <div className="grid grid-cols-1 md:grid-cols-3 gap-2 py-2 border-b border-gray-200 last:border-b-0">
        <dt className="text-sm font-medium text-gray-600 flex items-center">
          <span className="mr-2 text-indigo-600 text-lg">{checkboxIcon}</span>
          {title || 'Untitled Checkbox'}
        </dt>
        <dd className="md:col-span-2 text-sm text-gray-800">
          {/* Optionally display the actual string value if it's meaningful beyond checked/unchecked */}
          {isChecked && typeof dataNode === 'string' && dataNode.trim() !== '' ? `(${dataNode})` : ''}
          {!isChecked && (dataNode === undefined || dataNode === null || (typeof dataNode === 'string' && dataNode.trim() === '')) ? '[N/A]' : ''}
        </dd>
        {description && <p className="md:col-span-3 text-xs text-gray-500 mt-1">{description}</p>}
      </div>
    );
  }
  
  // Default rendering for other primitive types (textbox, string, boolean without specific element, etc.)
  let displayValue = "[N/A]";
  if (dataNode !== undefined && dataNode !== null) {
    if (type === 'boolean') { // Handles booleans not explicitly 'element: "checkbox"'
      displayValue = dataNode ? 'Yes' : 'No';
    } else if (schemaNode.enum && Array.isArray(schemaNode.enum) && schemaNode.enum.includes(dataNode)) {
        displayValue = String(dataNode); 
    } else {
      displayValue = String(dataNode);
    }
  }
  
  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-2 py-2 border-b border-gray-200 last:border-b-0">
      <dt className="text-sm font-medium text-gray-600 break-words">{title || 'Untitled Field'}:</dt>
      <dd className="md:col-span-2 text-sm text-gray-800 break-words">{displayValue}</dd>
      {description && schemaNode.type !== 'object' && <p className="md:col-span-3 text-xs text-gray-500 mt-1">{description}</p>}
    </div>
  );
};

function App() {
  const [renderMode, setRenderMode] = useState('simple'); 
  const [currentSchema, setCurrentSchema] = useState(schemaWithConditions);
  const [currentData, setCurrentData] = useState(sampleData);

  return (
    <div className="container mx-auto p-4 md:p-8 font-sans bg-gray-50 min-h-screen">
      <header className="mb-8 text-center">
        <h1 className="text-4xl font-bold text-indigo-600">Schema-Driven UI Renderer</h1>
        <p className="text-lg text-gray-600 mt-2">Displaying plan details based on JSON schema.</p>
      </header>

      <div className="mb-6 p-4 bg-white rounded-lg shadow-md">
        <h2 className="text-xl font-semibold mb-3 text-gray-700">Render Mode</h2>
        <div className="flex space-x-4">
          <button
            onClick={() => setRenderMode('simple')}
            className={`px-6 py-2 rounded-md font-medium transition-colors
                        ${renderMode === 'simple' ? 'bg-indigo-600 text-white shadow-lg' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'}`}
          >
            Simple Mode (Show All)
          </button>
          <button
            onClick={() => setRenderMode('conditional')}
            className={`px-6 py-2 rounded-md font-medium transition-colors
                        ${renderMode === 'conditional' ? 'bg-green-600 text-white shadow-lg' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'}`}
          >
            Conditional Mode (Use Visibility Rules)
          </button>
        </div>
         <p className="text-sm text-gray-500 mt-3">
            <strong>Simple Mode:</strong> Displays all fields defined in the schema. Missing data shown as "[N/A]".<br/>
            <strong>Conditional Mode:</strong> Uses <code>x-ui-visible-if</code> rules in the schema to show/hide fields based on data values.
        </p>
      </div>
      
      <main>
        {Object.keys(currentSchema).map(schemaKey => (
          <SchemaField
            key={schemaKey}
            schemaNode={currentSchema[schemaKey]} 
            dataNode={currentData && currentData[schemaKey]}
            fullData={currentData} 
            renderMode={renderMode}
            pathPrefix="" 
          />
        ))}
      </main>

      <footer className="mt-12 text-center text-sm text-gray-500">
        <p>&copy; {new Date().getFullYear()} Dynamic Renderer Inc. All rights reserved.</p>
      </footer>
    </div>
  );
}

export default App;









package com.example.demo.controller;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.io.File;

@RestController
public class MigrationController {

    @Autowired
    private ObjectMapper objectMapper;

    @PostMapping("/migrate-plan")
    public JsonNode migratePlan(
            @RequestBody JsonNode planRequest,
            @RequestParam String fromVersion,
            @RequestParam String toVersion
    ) throws Exception {
        // 1. Load schemas
        JsonNode fromSchema = objectMapper.readTree(new File("src/main/resources/schema/plan-schema-" + fromVersion + ".json"));
        JsonNode toSchema = objectMapper.readTree(new File("src/main/resources/schema/plan-schema-" + toVersion + ".json"));

        // 2. Migrate
        JsonNode migrated = migrateNode(planRequest, fromSchema.get("properties"), toSchema.get("properties"));

        // 3. Optionally update version field
        if (migrated instanceof com.fasterxml.jackson.databind.node.ObjectNode) {
            ((com.fasterxml.jackson.databind.node.ObjectNode) migrated).put("planCreationVersion", "plan-schema-" + toVersion);
        }

        return migrated;
    }

    // Recursive migration logic
    private JsonNode migrateNode(JsonNode plan, JsonNode fromProps, JsonNode toProps) {
        com.fasterxml.jackson.databind.node.ObjectNode result = objectMapper.createObjectNode();

        toProps.fieldNames().forEachRemaining(field -> {
            if (plan.has(field)) {
                if (toProps.get(field).has("properties")) {
                    result.set(field, migrateNode(
                            plan.get(field),
                            fromProps != null && fromProps.has(field) ? fromProps.get(field).get("properties") : null,
                            toProps.get(field).get("properties")
                    ));
                } else {
                    result.set(field, plan.get(field));
                }
            } else {
                // Field is new in target schema
                result.set(field, null);
            }
        });

        // No need to remove fields not in toProps, as only toProps fields are added
        return result;
    }
} 
