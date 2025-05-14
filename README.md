package com.example.demo.controller;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.JsonNodeFactory;
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
