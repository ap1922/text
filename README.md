-->service class:
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

@Service
public class DocumentService {

    @Autowired
    private DocumentRepository documentRepository;

    public List<Document> getAllFiltered(String documentType, String category, String name, String created) {
        // Convert the input created date to the format stored in the database
        SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MM-yyyy");
        String formattedCreatedDate = null;
        try {
            Date date = dateFormat.parse(created);
            formattedCreatedDate = dateFormat.format(date);
        } catch (Exception e) {
            // Handle parsing exception if needed
        }

        return documentRepository.findAllByDocumentTypeAndCategoryAndNameAndCreated(documentType, category, name, formattedCreatedDate);
    }
}

add specification :
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.Date;

public class DocumentSpecification {

    public static Specification<Document> createdDateEquals(String date) {
        return (Root<Document> root, CriteriaQuery<?> query, CriteriaBuilder builder) -> {
            if (date != null && !date.isEmpty()) {
                Date parsedDate = new SimpleDateFormat("dd-MM-yyyy").parse(date);
                Date nextDay = Date.from(parsedDate.toInstant().plus(1, ChronoUnit.DAYS));

                return builder.and(
                        builder.greaterThanOrEqualTo(root.get("created"), parsedDate),
                        builder.lessThan(root.get("created"), nextDay)
                );
            }
            return null;
        };
    }
}

in service:
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.List;

@Service
public class DocumentService {

    @Autowired
    private DocumentRepository documentRepository;

    public List<Document> getAllFiltered(String documentType, String category, String name, String created) {
        Specification<Document> spec = Specification.where(null);

        if (documentType != null && !documentType.isEmpty()) {
            spec = spec.and((root, query, builder) -> builder.equal(root.get("documentType"), documentType));
        }

        if (category != null && !category.isEmpty()) {
            spec = spec.and((root, query, builder) -> builder.equal(root.get("category"), category));
        }

        if (name != null && !name.isEmpty()) {
            spec = spec.and((root, query, builder) -> builder.equal(root.get("name"), name));
        }

        spec = spec.and(DocumentSpecification.createdDateEquals(created));

        return documentRepository.findAll(spec);
    }
}
in controller :
@GetMapping("/search")
public List<Document> searchDocuments(
        @RequestParam(required = false) String documentType,
        @RequestParam(required = false) String category,
        @RequestParam(required = false) String name,
        @RequestParam(required = false) String created
) {
    return documentService.getAllFiltered(documentType, category, name, created);
}





